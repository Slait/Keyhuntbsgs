Instructions for the mode -m FastECDLP
=================================

This file describes the experimental FastECDLP mode added to keyhunt.
The mode is designed to test the FastECDLP/BSGS-like search algorithm on
the same public keys and ranges that regular BSGS works with.

Important:
FastECDLP in the current implementation is an experimental mode for comparison.
c -m bsgs. He already knows how to build a baby-step T1 table, use sorted or
cuckoo lookup backend, parallelize giant loop by t, use batch32
to speed up giant points, as well as save/load cache T1/T2.

Basic launch
--------------

For example, the file test.txt Contains 037d5b40e63c41a0b70d499a41c8f256569bd4bf169b5e02d668bb0498c7178cea

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16

The same thing can be written in lowercase letters.:

  keyhunt.exe -m fastecdlp -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16

Basic parameters:

  -m FastECDLP
    Enables the FastECDLP experimental mode.

  -f test.txt
    A file with public keys. The format is the same as for BSGS:
compressed public key with a length of 66 hex characters or uncompressed public key
    130 hex characters long.

  -r START:END
Range of private keys in hex.

  -t N
    The number of threads. In FastECDLP, the giant loop is divided between threads.

  -s 0
    Disables the periodic output of regular keyhunt statistics. For FastECDLP
    The final metrics are printed at the end of the launch.


What does FastECDLP do?
--------------------

For each public key Q and range [start, end], the mode counts:

  Q' = Q - start*G

Then it looks for offset x such that:

  Q' = x*G

If offset is found, the private key is:

  privkey = start + x

The algorithm divides the search into:

  T1 / baby steps
    The table of small steps i*G.

  giant loop
    Sequential checks Q' - j*B, where B = 2^l1*G.

For x-coordinate matches, both candidates are checked.:

  j*B + i
  j*B - i

The final check is always done by recalculating the public key, so a false
match in the lookup table should not give a false found key.


--fe-l1 N
---------

Manual size selection of baby-step table T1.

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-l1 20

Acceptable range:

  10..40

What does l1 mean?:

  baby_count = 2^l1

The larger the l1:

  more T1 in memory and on disk;
  It takes longer to create T1;
  fewer giant steps;
  the search speed is potentially higher if there is enough RAM and the cache does not
slow down too much.

Examples of sizes for the sorted backend:

  --fe-l1 20
    baby_count = 1,048,576
    T1 is approximately 16 MB

  --fe-l1 24
    baby_count = 16,777,216
    T1 is approximately 256 MB

  --fe-l1 27
    baby_count = 134,217,728
    T1 is about 2 GB

An example for a table of about 1.8-2.0 GB:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 27


--fe-t1-mb N
------------

Automatically selects l1 for the approximate size of T1 in megabytes.

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-t1-mb 1800

What does:

  the program evaluates which l1 will give T1 approximately the specified size.;
  The selected l1 is printed at the start of the startup.;
  if --fe-l1 is specified at the same time, then --fe-l1 has priority.

Important:
--fe-t1-mb sets the size of the T1 lookup table FastECDLP, not the bloom filter.
There is currently no bloom filter in FastECDLP mode, as in -m bsgs. Here T1 is
the baby steps table.

Example:

  --fe-backend sorted --fe-t1-mb 1800

L1 will usually select about 30, because sorted T1 stores about 16 bytes per
element.


--fe-backend sorted|cuckoo
--------------------------

Selects the backend for T1 lookup.

The sorted example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 20

Example of a cuckoo:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20

sorted:

  T1 is stored as a key/index array;
  after creation, it is sorted;
  the search goes through binary search;
  memory is approximately 16 bytes per element;
  reliable fallback backend.

cuckoo:

  T1 is built as a cuckoo hash table;
  search is usually faster than binary search;
  The memory in the current implementation is approximately 22 bytes per element.;
  if the cuckoo table is not built, the program automatically rolls back to
  sorted backend and prints a warning.

Practical meaning:

  sorted is usually more memory-efficient;
  cuckoo can be faster on lookup, but uses more RAM.


--fe-t2-limit N
---------------

Sets the giant steps limit at which FastECDLP will build a T2 cache in advance
and use batch32 over T2.

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-l1 10 --fe-t2-limit 100

If:

  giant_count <= --fe-t2-limit

then the program builds/loads T2:

  filters/FastECDLP/t2_l*_g*.bin

If:

  giant_count > --fe-t2-limit

then the program runs in T2=stream mode, without storing the entire T2 in memory.

Default value:

  1048576

Disable T2 cache:

  --fe-t2-limit 0

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-l1 20 --fe-t2-limit 0

Important:
Even with T2=stream, the current implementation uses batch32 for giant points.
That is, stream mode is already accelerated and does not make a single AddDirect for each step.


--fe-max-giants N
-----------------

Limits the number of giant steps. It is used for benchmark tests, so
as not to wait for a full pass of a huge range.

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

What does:

  the program will count only the first N giant steps;
  if the key is not in this part, the program will terminate.;
  The resulting metrics will show the lookup/generation rate on the selected segment.

This is useful for comparing options.:

  sorted vs cuckoo;
  different --FE-l1;
  different -t;
  T2=batch32 vs T2=stream.

Important:
For a real complete search, do not specify --fe-max-giants.


--fe-profile
------------

Includes detailed FastECDLP hot path metrics.

Example:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 --fe-profile

Additionally prints the line:

  FastECDLP profile:

Main fields:

  add32_ns_call
    The average time of one AddDirect32 batch call.

  offset_ns_lookup
    The average offset calculation time per lookup.

  xkey_ns_lookup
    The average extraction time of a 64-bit xkey per lookup.

  backend_ns_lookup
    The average lookup time in the sorted/cuckoo backend.

  check_key_ns_lookup
    The average total check_key time per lookup.

  *_thread_share
    The share of thread-time relative to wall-time. In a multithreaded startup, there may be
    more than 100% is normal. Use this as an indicator of hot spots,
not as a regular percentage.

Important:
--fe-profile adds overhead due to timers and atomic counters. For the final
run benchmark without --fe-profile, and enable the profile only for
bottleneck analysis.


Cache T1/T2
-----------

FastECDLP cache is created automatically in the folder:

  filters/FastECDLP/

T1 cache:

  filters/FastECDLP/t1_l10_b1024.bin
  filters/FastECDLP/t1_l20_b1048576.bin
  filters/FastECDLP/t1_l27_b134217728.bin

Name format:

  t1_l{l1}_b{baby_count}.bin

T2 cache:

  filters/FastECDLP/t2_l10_g21.bin
  filters/FastECDLP/t2_l20_g100000.bin

Name format:

  t2_l{l1}_g{giant_count}.bin

When the cache is created:

  if there is no file;
  if the l1/baby_count/giant_count parameters are different;
  if the file is unreadable or does not match the expected size.

When the cache is loaded:

  if the file already exists and matches the current settings.

The output will show:

  [+] FastECDLP T1 cache saved: ...
  [+] FastECDLP T1 cache loaded: ...
  [+] FastECDLP T2 cache saved: ...
  [+] FastECDLP T2 cache loaded: ...

FastECDLP Metrics
-----------------

Output example:

  [+] FastECDLP l1=20 l2=17 baby=1048576 giant=100000/1529008357377 T1=16.00 MB backend=cuckoo T2=stream 0.00 MB
  [+] FastECDLP metrics: precompute=0.155s search=0.007s lookups=100000 lookup_ns=72.69 table_hits=0 candidates=0 verified=0 hit_rate=0.000000% candidate_rate=0.000000% batch32=3040
  [+] FastECDLP estimated speed: 0.014 Pkeys/s (14425312972899 keys/s)

Fields:

  l1
    The size of the baby-step part. baby_count = 2^l1.

  l2
    Approximate size of the giant-step part in bits.

  baby
Number of baby steps.

  giant=A/B
A = how many giant steps will actually be tested in this run.
    B = how many giant steps are needed for the full range.
    If --fe-max-giants is specified, A may be less than B.

  T1
    The approximate size of T1 in memory.

  backend
    sorted or cuckoo.

  T2
    batch32 or stream.

  precompute
    Preparation time T1/T2/cache/backend.

  search
    The time of the giant loop search.

  lookups
    The number of requests to T1.

  lookup_ns
    The average time per giant lookup, taking into account the generation of giant points.

  table_hits
    The number of matches in T1.

  candidates
    The number of candidates who passed before the final check.

  verified
    The number of confirmed candidates.

  hit_rate
    table_hits / lookups.

  candidate_rate
    candidates / lookups.

  batch32
    The number of batch32 calls to generate giant points.

  estimated speed
    Speed rating in keys/s and Pkeys/s.


Comparison with BSGS bloom/fuse
---------------------------

Regular BSGS:

  keyhunt.exe -m bsgs -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -k 128 -S -s 1 -x bloom -t 16

  keyhunt.exe -m bsgs -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -k 128 -S -s 1 -x fuse -t 16

In BSGS bloom/fuse, filters are created and loaded via -S.

Ways:

  filters/bloom/k128n0x100000000000/
  filters/FUSE/k128n0x100000000000/

The size of bloom/fuse depends on:

  -n
    Defines N for BSGS. By default, N = 0x100000000000.

  -k
is a multiplier of M. The higher the -k, the more filters and RAM, but the less giant work.

  -x
    Backend type: bloom, fuse, xor, blocked.

For bloom, the current code uses a false-positive rate of 0.000001.

FastECDLP:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

To complete the search, remove:

  --fe-max-giants


How to get T1 about 1.8 GB
----------------------------

Option 1, automatic:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-t1-mb 1800

Option 2, manual:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend sorted --fe-l1 27

For sorted:

  l1=27 approximately 2.0 GB.

For cuckoo:

  l1=26 approximately 1.4 GB;
  l1=27 is approximately 2.8 GB in the current implementation.

If RAM is limited, it is better to start with:

  --fe-l1 24
  --fe-l1 25
  --fe-l1 26


It's important about "the bigger the filter, the faster"
-----------------------------------------

The idea is partially correct, but with caveats.

In BSGS:

  larger -K increases M and filters;
  giant loop is getting shorter;
  but RAM is increasing and the preparation/loading time is increasing.

In FastECDLP:

  a larger T1 increases baby_count;
  full_giant_count is decreasing;
  The lookup table is getting bigger;
  cache/RAM pressure is growing;
  T1 that is too large may start to slow down due to memory.

Therefore, the optimum must be sought by tests.

Recommended order of FastECDLP tests:

  1. Check correctness on a small range:

     keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-backend sorted --fe-l1 10

  2. Check out the cuckoo:

     keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -t 4 --fe-backend cuckoo --fe-l1 10

  3. Benchmark on a large range without a full pass:

     keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -t 16 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000

  4. Increase l1:

     --fe-l1 22
     --fe-l1 24
     --fe-l1 26
     --fe-l1 27

  5. Compare estimated speed, lookup_ns, precompute and RAM.


Practical commands
--------------------

A small test sorted:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -s 0 --fe-backend sorted --fe-l1 10 -t 4

The little cuckoo test:

  keyhunt.exe -m FastECDLP -f test.txt -r 1:5000 -s 0 --fe-backend cuckoo --fe-l1 10 -t 4

Benchmark sorted:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend sorted --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Benchmark cuckoo:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend cuckoo --fe-l1 20 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Example T1 is about 1.8 GB:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 0 --fe-backend sorted --fe-t1-mb 1800 --fe-t2-limit 0 --fe-max-giants 100000 -t 16

Full launch without a benchmark limiter:

  keyhunt.exe -m FastECDLP -f test.txt -r 80a15e21125e3806:96e15e21125e3806 -s 1 --fe-backend sorted --fe-l1 27 --fe-t2-limit 0 -t 16