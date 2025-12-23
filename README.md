# KeyHunt - BSGS Search [RUS](README_ru.md)

KeyHunt is a high-performance tool for searching Bitcoin private keys using the Baby-Step Giant-Step (BSGS) algorithm. This version includes significant optimizations for modern processors, improved memory management, and flexible search settings.

![ket hunt](https://github.com/Slait/Keyhuntbsgs/blob/main/keyhunt.png?raw=true)

## Key Features and Changes
- **Higher speed**: Compared to the original Keyhunt, the speed is 20-40% faster for BSGS, with faster Bloom filter creation and faster Bloom filter loading.
- **BSGS Algorithm**: Efficient implementation of the algorithm for solving the discrete logarithm problem.
- **Bloom Filters**: Uses memory-efficient Bloom filters to store precomputed tables.
- **Parallel Processing**: 
  - **Parallel Bloom Filter Loading**: Filters are loaded in parallel across multiple threads, drastically reducing startup time.
  - **Multithreading**: Uses all available CPU cores for search by default.
- **Optimized Hashing**: 
  - Switch to **XXH64** for Bloom filter checksums and internal hashing, replacing slower functions.
  - File checksums are now verified via XXH64.
- **Search Flexibility**:
  - **WIF Mask**: Search for private keys matching a specific WIF pattern (`-w` flag).
  - **Custom Ranges**: Ability to specify specific hexadecimal (HEX) search ranges.
  - **Progress Saving**: Progress is automatically saved to `checked-deep.txt` when using `-D` mode, allowing you to resume search from where it was interrupted.
- **Console Output**: Optional output of WIF keys to the console (`-wif 1`).
- **File Organization**: Bloom filters are now saved in a separate `filters/` folder with organized names.

## Usage
![ket hunt](https://github.com/Slait/Keyhuntbsgs/blob/main/keyhunt2.png?raw=true)
## Disclaimer
I created this tool as a universal tool for solving puzzles. I recommend everyone to use it ONLY for solving puzzles.

Some users asked me to add support for Ethereum and mini-keys, and I did. But again, I recommend using this program ONLY for solving puzzles.

## Download and Build
This program was developed in a Linux environment. If you are using Windows, I highly recommend using the WSL environment in Windows. It is available in the Microsoft Store.

Please install it on your system.
```bash
git
build-essential
libssl-dev
libgmp-dev
```

On Ubuntu-based systems, run the following commands to update your current environment and install the tools needed for compilation.

```bash
apt update && apt upgrade
apt install git -y
apt install build-essential -y
apt install libssl-dev -y
apt install libgmp-dev -y
```

To clone the repository:

```bash
git clone https://github.com/Slait/Keyhuntbsgs.git
```

Don't forget to navigate to the keyhunt directory (but I'm not here to teach you Linux commands).

```bash
cd keyhunt
```

### Build
Use `Makefile` to compile. You can specify the target architecture for optimization:
```bash
# Auto-detect architecture (recommended)
make

# Explicit architecture specification (e.g., for AMD EPYC 9654)
make ARCH=znver4
```
Supported architectures: `native` (default)
```
# AMD:
   znver4        - AMD EPYC 9654, Ryzen 7000 series
   znver3        - Ryzen 5000 series
   znver2        - Ryzen 3000 series

# Intel:
   arrowlake     - Intel Core Ultra 200 series (e.g. 265KF) (Requires GCC 14+)
   raptorlake    - Intel 13th Gen & 14th Gen (Core i9-13900K, i9-14900K) (Requires GCC 13+)
   alderlake     - Intel 12th Gen (Core i9-12900K)
   skylake       - Older Intel CPUs

# Generic:
   native        - Auto-detect host CPU (Best for local compilation)
   x86-64-v3     - Generic modern CPUs (AVX2)
   x86-64-v4     - Generic very modern CPUs (AVX512)
```

### Normal BSGS Search
Run a standard BSGS search in the default range or random start.
```bash
./keyhunt -m bsgs -t 8 -f test.txt -S -s 10
```
* `-m bsgs`: Set BSGS mode.
* `-t 8`: Use 8 threads (if omitted, uses all cores).
* `-f test.txt`: File containing public keys to search for.
* `-S`: Save bloom filter to `filters` folder.
* `-s 10`: Output search info, speed, and time every 10 seconds to console. If `-s 0` is specified, output is hidden. Any number of seconds can be specified.


### WIF Output
Enable printing found WIF keys to the console (default is off/0).
```bash
./keyhunt -m bsgs -f test.txt -S -s 10 -wif 1
```
* `-wif 1`: Print WIF key information to the string.

### WIF Mask Search
Search for a private key that matches a partial WIF string (mask). This generates a `deep.txt` file with ranges and processes them.
```bash
./keyhunt -m bsgs -w KwDiBf89QgGbjEhKnhXJuH_LrciVrZi_qYwk________________

Warning, offset is not implemented yet. Suitable for searching for the last characters or unknowns. The program creates a deep.txt file with ranges and processes them.
```
* `-w <mask_pattern>`: Specify WIF pattern.

### Custom Range
Search within a specific hexadecimal range `Start:End`.
```bash
./keyhunt -m bsgs -r 20000000000000000:3ffffffffffffffff -f test.txt -S -s 10
```
* `-r 20000000000000000:3ffffffffffffffff`: You can specify the search range manually.

### Continue Search (Deep Mode)
Use `-D` to process ranges from a file (default `deep.txt` or created by `-w` flag). Progress is tracked in `checked-deep.txt`.
```bash
./keyhunt -m bsgs -D deep.txt
```
* `-D deep.txt`: You can specify multiple ranges in the `deep.txt` file; after checking a range, the program automatically moves to the next one and saves the completed range to `checked-deep.txt`.


### Filter Generation and Storage
Bloom filters are generated automatically if they do not exist. They are saved in the `filters/` directory if you specify the `-S` key, to avoid cluttering the working folder.
**File Names**: Filter files are named sequentially (`keyhunt_bsgs_1...`, `_2`, etc.) for better order.

**Note**: Loading time is significantly improved due to parallelization.

## Command Line Arguments

| Flag         | Description                                                                       |
|--------------|-----------------------------------------------------------------------------------|
| `-m bsgs`    | **Required**. Sets BSGS mode.                                                     |
| `-t <n>`     | Number of threads (default: auto-detect all cores).                               |
| `-wif <0/1>` | Enable (1) or disable (0) WIF output to console.                                  |
| `-w <mask>`  | Search using WIF mask.                                                            |
| `-r <s:e>`   | Set search range (HEX format start:end).                                          |
| `-D <file>`  | Run in "Deep" mode using a range file (e.g., `deep.txt`).                         |
| `-n <val>`   | Set custom N value (Total points). Default 0x100000000000                         |
| `-k <val>`   | Set factor K. I recommend using values from the table below                       |
| `-d`         | Enable debug mode.                                                                |


Correct values for n and maximum k for specific bits
-------------------------------------------------
| Size    | bits | `-k` (dec) | `-n` (hex)      |
| ------- | ---- | ---------- | --------------- |
| 2 GB    | 44   | 128        | 0x100000000000  |
| 4 GB    | 44   | 256        | 0x100000000000  |
| 8 GB    | 44   | 512        | 0x100000000000  |
| 16 GB   | 44   | 1024       | 0x100000000000  |
| 32 GB   | 44   | 2048       | 0x100000000000  |
| 64 GB   | 44   | 4096       | 0x100000000000  |
| 128 GB  | 44   | 8192       | 0x100000000000  |
| 256 GB  | 46   | 16384      | 0x400000000000  |
| 384 GB  | 46   | 24576      | 0x400000000000  |
| 512 GB  | 48   | 32768      | 0x1000000000000 |
| 640 GB  | 48   | 40960      | 0x1000000000000 |
| 768 GB  | 48   | 49152      | 0x1000000000000 |
| 896 GB  | 48   | 57344      | 0x1000000000000 |
| 1 TB    | 48   | 65536      | 0x1000000000000 |
| 1152 GB | 50   | 73728      | 0x4000000000000 |
| 1280 GB | 50   | 81920      | 0x4000000000000 |
| 1408 GB | 50   | 90112      | 0x4000000000000 |
| 1536 GB | 50   | 98304      | 0x4000000000000 |
| 1664 GB | 50   | 106496     | 0x4000000000000 |
| 1792 GB | 50   | 114688     | 0x4000000000000 |
| 1920 GB | 50   | 122880     | 0x4000000000000 |
| 2 TB    | 50   | 131072     | 0x4000000000000 |

-------------------------------------------------

