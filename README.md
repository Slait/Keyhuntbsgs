# keyhunt

Tool for hunt privatekeys for crypto currencies that use secp256k1 elliptic curve

Language:
- English (this file)
- Русский: ./README_ru.md

Post: https://bitcointalk.org/index.php?topic=5322040.0


# Recommended BSGS (-k / -n) up to 1024GB RAM

Notes:
- For BSGS, `-n` sets `N` (accepts decimal or `0xHEX`). `N` must have an **exact** square root and `M = sqrt(N)` must be divisible by 1024.
- `-k` increases speed but also increases RAM/file sizes.
- Prefer power-of-two values for `-k` and `-n`.

Baseline used by this project:
- Default `-n` is `0x100000000000`.
- With `-k 4096` and default `-n`, RAM usage is about ~64GB.

If you want to scale RAM beyond ~64GB while keeping `-k` in the safe/default range (<= 4096), increase `-n`:

| RAM target | Suggested `-k` | Suggested `-n` |
| --- | ---: | --- |
| 8 GB | 512 | 0x100000000000 |
| 16 GB | 1024 | 0x100000000000 |
| 32 GB | 2048 | 0x100000000000 |
| 64 GB | 4096 | 0x100000000000 |
| 128 GB | 4096 | 0x400000000000 |
| 256 GB | 4096 | 0x1000000000000 |
| 512 GB | 4096 | 0x4000000000000 |
| 1024 GB | 4096 | 0x10000000000000 |

Always verify the actual memory usage in the program output (it prints bloom/table sizes) and adjust.


```

You need to add `-t numberThreads` to get better speed

- Run against Puzzle 125 (bsgs mode)

```
./keyhunt -m bsgs -f tests/125.txt -b 125 -q -s 10 -R
```

You need to add `-t numberThreads` and `-k factor` to get better speed

## Free Code

This code is free of charge, see the licence for more details. https://github.com/albertobsd/keyhunt/blob/main/LICENSE

Although this project is a hobby for me, it still involves a considerable amount of work.
If you would like to support this project, please consider donating at https://github.com/albertobsd/keyhunt#donations.


# Disclaimer

I made this tool as a generic tool for the Puzzles.
I recommend to everyone to stay in puzzles

Several of users request me to add support for ethereum and minikeys, I did it.
But again i recommend only use this program for puzzles.

## For regular users

Please read the CHANGELOG.md to see the new changes

# Download and build

This program was made in a linux environment.
if you are windows user i strongly recommend to use WSL enviroment on Windows.
it is available in the Microsoft store

Please install on your system

- git
- build-essential

for legacy version also you are going to need:

- libssl-dev
- libgmp-dev

On Debian based systems, run this commands to update your current enviroment
and install the tools needed to compile it

```
apt update && apt upgrade
apt install git -y
apt install build-essential -y
apt install libssl-dev -y
apt install libgmp-dev -y
```

To clone the repository

```
git clone https://github.com/slait/keyhuntbsgs.git
```

don't forget change to the keyhunt directory (But i'm not here to teach you linux commands)

```
cd keyhunt
```

First compile:

```
make
```


and then execute with `-h` to see the help

```
./keyhunt -h
```



## bsgs mode (baby step giant step)

Keyhunt implement the BSGS algorithm to search privatekeys for a known public key.

The input file need to have a list of publickeys compress or uncompress those publickey can be mixed in the same file, one public key per line and any other word followed by an space is ignored example of the file:

```
043ffa1cc011a8d23dec502c7656fb3f93dbe4c61f91fd443ba444b4ec2dd8e6f0406c36edf3d8a0dfaa7b8f309b8f1276a5c04131762c23594f130a023742bdde # 0000000000000000000000000000000000800000000000000000100000000000
046534b9e9d56624f5850198f6ac462f482fec8a60262728ee79a91cac1d60f8d6a92d5131a20f78e26726a63d212158b20b14c3025ebb9968c890c4bab90bfc69 # 0000000000000000000000000000000000800000000000000000200000000000
```

This example contains 2 publickeys followed by his privatekey just to test the correct behavior of the application.

*Don't load more than 100 or 1000 publickeys* if you lad more than it will take a long long time in update the speed counter and the speed will be very low.

btw any word followed by and space after the publickey is ignored the file can be only the publickeys:

```
043ffa1cc011a8d23dec502c7656fb3f93dbe4c61f91fd443ba444b4ec2dd8e6f0406c36edf3d8a0dfaa7b8f309b8f1276a5c04131762c23594f130a023742bdde
046534b9e9d56624f5850198f6ac462f482fec8a60262728ee79a91cac1d60f8d6a92d5131a20f78e26726a63d212158b20b14c3025ebb9968c890c4bab90bfc69
```

### File creation

the bsgs mode `-m bsgs` now can create automatically the files needed to speed up the initial load process of keyhunt this is the bloom filters creation and the bp table creation.

To request to keyhunt to create those files automatically use `-S` Capital S for SAVE and READ files.
The 3 files needed for keyhunt can vary from size depending of your values of `-n` and `-k` , so make your test and stick to one combination of (n,k) values or you can end with hundreds of unnesesary files.

The 3 Files size are the same amount of memory used in runtime.

The files are created if they don't exist when you run the program the first time.

example of file creation:

```
./keyhunt -m bsgs -f tests/125.txt -R -b 125 -q -S -s 10
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Random mode
[+] Quiet thread output
[+] Stats output every 10 seconds
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 4194304 elements : 14.38 MB
[+] Bloom filter for 131072 elements : 0.88 MB
[+] Bloom filter for 4096 elements : 0.88 MB
[+] Allocating 0.00 MB for 4096 bP Points
[+] processing 4194304/4194304 bP points : 100%
[+] Making checkums .. ... done
[+] Sorting 4096 elements... Done!
[+] Writing bloom filter to file keyhunt_bsgs_4_4194304.blm .... Done!
[+] Writing bloom filter to file keyhunt_bsgs_6_131072.blm .... Done!
[+] Writing bP Table to file keyhunt_bsgs_2_4096.tbl .. Done!
[+] Writing bloom filter to file keyhunt_bsgs_7_4096.blm .... Done!
^C] Total 457396837154816 keys in 30 seconds: ~15 Tkeys/s (15246561238493 keys/s)
```

When we run the program for second time the files are now readed and the bP Points processing is omitted:

```
./keyhunt -m bsgs -f tests/125.txt -R -b 125 -q -S -s 10
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Random mode
[+] Quiet thread output
[+] Stats output every 10 seconds
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 4194304 elements : 14.38 MB
[+] Bloom filter for 131072 elements : 0.88 MB
[+] Bloom filter for 4096 elements : 0.88 MB
[+] Allocating 0.00 MB for 4096 bP Points
[+] Reading bloom filter from file keyhunt_bsgs_4_4194304.blm .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_6_131072.blm .... Done!
[+] Reading bP Table from file keyhunt_bsgs_2_4096.tbl .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_7_4096.blm .... Done!
^C
```

All the next examples were made with the `-S` option I just ommit that part of the output to avoid confutions use `-S` if you want, but remember with a great `-n` there must also come great files

### Examples

To try to find those privatekey this is the line of execution:

```
time ./keyhunt -m bsgs -f tests/test120.txt -b 120 -S
```

Output:

```
time ./keyhunt -m bsgs -f tests/test120.txt -b 120 -S
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Mode BSGS secuential
[+] Opening file tests/test120.txt
[+] Added 2 points from file
[+] Bit Range 120
[+] -- from : 0x800000000000000000000000000000
[+] -- to   : 0x1000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 4194304 elements : 14.38 MB
[+] Bloom filter for 131072 elements : 0.88 MB
[+] Bloom filter for 4096 elements : 0.88 MB
[+] Allocating 0.00 MB for 4096 bP Points
[+] Reading bloom filter from file keyhunt_bsgs_4_4194304.blm .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_6_131072.blm .... Done!
[+] Reading bP Table from file keyhunt_bsgs_2_4096.tbl .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_7_4096.blm .... Done!
[+] Thread Key found privkey 800000000000000000100000000000
[+] Publickey 043ffa1cc011a8d23dec502c7656fb3f93dbe4c61f91fd443ba444b4ec2dd8e6f0406c36edf3d8a0dfaa7b8f309b8f1276a5c04131762c23594f130a023742bdde
[+] Thread Key found privkey 800000000000000000200000000000
[+] Publickey 046534b9e9d56624f5850198f6ac462f482fec8a60262728ee79a91cac1d60f8d6a92d5131a20f78e26726a63d212158b20b14c3025ebb9968c890c4bab90bfc69
All points were found

real    0m3.632s
user    0m3.619s
sys     0m0.000s
```

Test the puzzle 120 with the next publickey:

```
0233709eb11e0d4439a729f21c2c443dedb727528229713f0065721ba8fa46f00e
```

Line of execution in random mode `-R` or -B random

```./keyhunt -m bsgs -f tests/125.txt -b 125 -q -s 10 -R```

```./keyhunt -m bsgs -f tests/125.txt -b 125 -q -s 10 -B random```


Example Output:

```
[+] Version 0.2.230507 Satoshi Quest, developed by AlbertoBSD
[+] Quiet thread output
[+] Stats output every 10 seconds
[+] Random mode
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 4194304 elements : 14.38 MB
[+] Bloom filter for 131072 elements : 0.88 MB
[+] Bloom filter for 4096 elements : 0.88 MB
[+] Allocating 0.00 MB for 4096 bP Points
[+] processing 4194304/4194304 bP points : 100%
[+] Making checkums .. ... done
[+] Sorting 4096 elements... Done!
[+] Total 158329674399744 keys in 10 seconds: ~15 Tkeys/s (15832967439974 keys/s)
```

Good speed no? 15 Terakeys/s for one single thread

**^C] Total 158329674399744 keys in 10 seconds: ~15 Tkeys/s (15832967439974 keys/s)**

We can speed up our process selecting a bigger K value `-k value` btw the n value is the total length of item tested in the radom range, a bigger k value means more ram to be use:

Example:
```
./keyhunt -m bsgs -f tests/125.txt -b 125 -R -k 20 -S
```

Output:

```
./keyhunt -m bsgs -f tests/125.txt -b 125 -R -k 20 -S
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Random mode
[+] K factor 20
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0xfffff000000
[+] Bloom filter for 83886080 elements : 287.55 MB
[+] Bloom filter for 2621440 elements : 8.99 MB
[+] Bloom filter for 81920 elements : 0.88 MB
[+] Allocating 1.00 MB for 81920 bP Points
[+] processing 83886080/83886080 bP points : 100%
[+] Making checkums .. ... done
[+] Sorting 81920 elements... Done!
[+] Writing bloom filter to file keyhunt_bsgs_4_83886080.blm .... Done!
[+] Writing bloom filter to file keyhunt_bsgs_6_2621440.blm .... Done!
[+] Writing bP Table to file keyhunt_bsgs_2_81920.tbl .. Done!
[+] Writing bloom filter to file keyhunt_bsgs_7_81920.blm .... Done!
^C] Thread 0x1bbb290563ffcf38724482a45f2bed04  ~256 Tkeys/s (256259265658880 keys/s)
```

**~256 Terakeys/s for one single thread**

Note the value of N `0xfffff000000` with k = 20 this mean that the N value is less than the default value `0x100000000000` that is because k is not a 2^X number

if you want to more Speed use a bigger -k value like 128, it will use some 2 GB of RAM


```
./keyhunt -m bsgs -f tests/125.txt -b 125 -R -k 128 -S
```

Output

```
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Random mode
[+] K factor 128
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 536870912 elements : 1840.33 MB
[+] Bloom filter for 16777216 elements : 57.51 MB
[+] Bloom filter for 524288 elements : 1.80 MB
[+] Allocating 8.00 MB for 524288 bP Points
[+] Reading bloom filter from file keyhunt_bsgs_4_536870912.blm .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_6_16777216.blm .... Done!
[+] Reading bP Table from file keyhunt_bsgs_2_524288.tbl .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_7_524288.blm .... Done!
^C] Thread 0x1d0e05e7aaf9eca861fe0b2245579241   ~1 Pkeys/s (1292439268063095 keys/s)
```

**~1.2 Pkeys/s for one single thread**

OK at this point maybe you want to use ALL your RAM memory to solve the puzzle 125, just a bigger -k value

I already tested it with some **8 GB ** used with `-k 512` and I get **~46 Petakeys/s per thread.**

with **8** threads

`./keyhunt -m bsgs -f tests/125.txt -b 125 -R -k 512 -q -t 8 -s 10 -S`

Output:

```
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Random mode
[+] K factor 512
[+] Quiet thread output
[+] Threads : 8
[+] Stats output every 10 seconds
[+] Mode BSGS random
[+] Opening file tests/125.txt
[+] Added 1 points from file
[+] Bit Range 125
[+] -- from : 0x10000000000000000000000000000000
[+] -- to   : 0x20000000000000000000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 2147483648 elements : 7361.33 MB
[+] Bloom filter for 67108864 elements : 230.04 MB
[+] Bloom filter for 2097152 elements : 7.19 MB
[+] Allocating 32.00 MB for 2097152 bP Points
[+] Reading bloom filter from file keyhunt_bsgs_4_2147483648.blm .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_6_67108864.blm .... Done!
[+] Reading bP Table from file keyhunt_bsgs_2_2097152.tbl .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_7_2097152.blm .... Done!
^C] Total 2126103644397895680 keys in 110 seconds: ~19 Pkeys/s (19328214949071778 keys/s)
```
I get ~19 Petakeys/s total

Warning: the default n value have a maximun K of `4096` if that value is exceed the program can have an unknow behavior or suboptimal speed.
If you want to use a bigger K I recomend use a bigger N value `-n 0x400000000000` and half your K value.

Just as comparation with the BSGS program of JLP
Same publickeys and ranged used by his sample:

publickeys:
```
0459A3BFDAD718C9D3FAC7C187F1139F0815AC5D923910D516E186AFDA28B221DC994327554CED887AAE5D211A2407CDD025CFC3779ECB9C9D7F2F1A1DDF3E9FF8
04A50FBBB20757CC0E9C41C49DD9DF261646EE7936272F3F68C740C9DA50D42BCD3E48440249D6BC78BC928AA52B1921E9690EBA823CBC7F3AF54B3707E6A73F34
0404A49211C0FE07C9F7C94695996F8826E09545375A3CF9677F2D780A3EB70DE3BD05357CAF8340CB041B1D46C5BB6B88CD9859A083B0804EF63D498B29D31DD1
040B39E3F26AF294502A5BE708BB87AEDD9F895868011E60C1D2ABFCA202CD7A4D1D18283AF49556CF33E1EA71A16B2D0E31EE7179D88BE7F6AA0A7C5498E5D97F
04837A31977A73A630C436E680915934A58B8C76EB9B57A42C3C717689BE8C0493E46726DE04352832790FD1C99D9DDC2EE8A96E50CAD4DCC3AF1BFB82D51F2494
040ECDB6359D41D2FD37628C718DDA9BE30E65801A88A00C3C5BDF36E7EE6ADBBAD71A2A535FCB54D56913E7F37D8103BA33ED6441D019D0922AC363FCC792C29A
0422DD52FCFA3A4384F0AFF199D019E481D335923D8C00BADAD42FFFC80AF8FCF038F139D652842243FC841E7C5B3E477D901F88C5AB0B88EE13D80080E413F2ED
04DB4F1B249406B8BD662F78CBA46F5E90E20FE27FC69D0FBAA2F06E6E50E536695DF83B68FD0F396BB9BFCF6D4FE312F32A43CF3FA1FE0F81DF70C877593B64E0
043BD0330D7381917F8860F1949ACBCCFDC7863422EEE2B6DB7EDD551850196687528B6D2BC0AA7A5855D168B26C6BAF9DDCD04B585D42C7B9913F60421716D37A
04332A02CA42C481EAADB7ADB97DF89033B23EA291FDA809BEA3CE5C3B73B20C49C410D1AD42A9247EB8FF217935C9E28411A08B325FBF28CC2AF8182CE2B5CE38
04513981849DE1A1327DEF34B51F5011C5070603CA22E6D868263CB7C908525F0C19EBA6BD2A8DCF651E4342512EDEACB6EA22DA323A194E25C6A1614ABD259BC0
04D4E6FA664BD75A508C0FF0ED6F2C52DA2ADD7C3F954D9C346D24318DBD2ECFC6805511F46262E10A25F252FD525AF1CBCC46016B6CD0A7705037364309198DA1
0456B468963752924DBF56112633DC57F07C512E3671A16CD7375C58469164599D1E04011D3E9004466C814B144A9BCB7E47D5BACA1B90DA0C4752603781BF5873
04D5BE7C653773CEE06A238020E953CFCD0F22BE2D045C6E5B4388A3F11B4586CBB4B177DFFD111F6A15A453009B568E95798B0227B60D8BEAC98AF671F31B0E2B
04B1985389D8AB680DEDD67BBA7CA781D1A9E6E5974AAD2E70518125BAD5783EB5355F46E927A030DB14CF8D3940C1BED7FB80624B32B349AB5A05226AF15A2228
0455B95BEF84A6045A505D015EF15E136E0A31CC2AA00FA4BCA62E5DF215EE981B3B4D6BCE33718DC6CF59F28B550648D7E8B2796AC36F25FF0C01F8BC42A16FD9
```

set range

```
-r 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e0000000000000000:49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5effffffffffffffff
```

the n value to get the same baby step table:


```
-n 1152921504606846976
```

number of threads

```
-t 6
```

Hidding the speed:

```
-s 0
```

command:

```
time ./keyhunt -m bsgs -t 6 -f tests/in.txt -r 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e0000000000000000:49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5effffffffffffffff -n 0x1000000000000000 -M -s 0
```

Output:
```
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Threads : 6
[+] Matrix screen
[+] Turn off stats output
[+] Mode BSGS secuential
[+] Opening file tests/in.txt
[+] Added 16 points from file
[+] Range
[+] -- from : 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e0000000000000000
[+] -- to   : 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5effffffffffffffff
[+] N = 0x1000000000000000
[+] Bloom filter for 1073741824 elements : 3680.00 MB
[+] Bloom filter for 53687092 elements : 184.03 MB
[+] Allocating 819.00 MB for 53687092 bP Points
[+] processing 1073741824/1073741824 bP points : 100%
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e0000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e2000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e1000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e3000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e4000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e5000000000000000
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e5698aaab6cac52b3
[+] Publickey 0404a49211c0fe07c9f7c94695996f8826e09545375a3cf9677f2d780a3eb70de3bd05357caf8340cb041b1d46c5bb6b88cd9859a083b0804ef63d498b29d31dd1
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e59c839258c2ad7a0
[+] Publickey 040b39e3f26af294502a5be708bb87aedd9f895868011e60c1d2abfca202cd7a4d1d18283af49556cf33e1ea71a16b2d0e31ee7179d88be7f6aa0a7c5498e5d97f
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e38160da9ebeaecd7
[+] Publickey 04db4f1b249406b8bd662f78cba46f5e90e20fe27fc69d0fbaa2f06e6e50e536695df83b68fd0f396bb9bfcf6d4fe312f32a43cf3fa1fe0f81df70c877593b64e0
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e54cad3cfbc2a9c2b
[+] Publickey 04332a02ca42c481eaadb7adb97df89033b23ea291fda809bea3ce5c3b73b20c49c410d1ad42a9247eb8ff217935c9e28411a08b325fbf28cc2af8182ce2b5ce38
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e0d5eccc38d0230e6
[+] Publickey 04513981849de1a1327def34b51f5011c5070603ca22e6d868263cb7c908525f0c19eba6bd2a8dcf651e4342512edeacb6ea22da323a194e25c6a1614abd259bc0
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e2452dd26bc983cd5
[+] Publickey 04b1985389d8ab680dedd67bba7ca781d1a9e6e5974aad2e70518125bad5783eb5355f46e927a030db14cf8d3940c1bed7fb80624b32b349ab5a05226af15a2228
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e6000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e7000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e8000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e9000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ea000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5eb000000000000000
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ebb3ef3883c1866d4
[+] Publickey 0459a3bfdad718c9d3fac7c187f1139f0815ac5d923910d516e186afda28b221dc994327554ced887aae5d211a2407cdd025cfc3779ecb9c9d7f2f1a1ddf3e9ff8
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5eb5abc43bebad3207
[+] Publickey 04a50fbbb20757cc0e9c41c49dd9df261646ee7936272f3f68c740c9da50d42bcd3e48440249d6bc78bc928aa52b1921e9690eba823cbc7f3af54b3707e6a73f34
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e765fb411e63b92b9
[+] Publickey 04837a31977a73a630c436e680915934a58b8c76eb9b57a42c3c717689be8c0493e46726de04352832790fd1c99d9ddc2ee8a96e50cad4dcc3af1bfb82d51f2494
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e7d0e6081c7e0e865
[+] Publickey 040ecdb6359d41d2fd37628c718dda9be30e65801a88a00c3c5bdf36e7ee6adbbad71a2a535fcb54d56913e7f37d8103ba33ed6441d019d0922ac363fcc792c29a
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e79d808cab1decf8d
[+] Publickey 043bd0330d7381917f8860f1949acbccfdc7863422eee2b6db7edd551850196687528b6d2bc0aa7a5855d168b26c6baf9ddcd04b585d42c7b9913f60421716d37a
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e7c43b8e079ae7278
[+] Publickey 0456b468963752924dbf56112633dc57f07c512e3671a16cd7375c58469164599d1e04011d3e9004466c814b144a9bcb7e47d5baca1b90da0c4752603781bf5873
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e8d63ef128ef66b42
[+] Publickey 04d5be7c653773cee06a238020e953cfcd0f22be2d045c6e5b4388a3f11b4586cbb4b177dffd111f6a15a453009b568e95798b0227b60d8beac98af671f31b0e2b
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5e7ad38337c7f173c7
[+] Publickey 0455b95bef84a6045a505d015ef15e136e0a31cc2aa00fa4bca62e5df215ee981b3b4d6bce33718dc6cf59f28b550648d7e8b2796ac36f25ff0c01f8bc42a16fd9
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ec000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ed000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ee000000000000000
[+] Thread 0x49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ef000000000000000
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ec737344ca673ce28
[+] Publickey 0422dd52fcfa3a4384f0aff199d019e481d335923d8c00badad42fffc80af8fcf038f139d652842243fc841e7c5b3e477d901f88c5ab0b88ee13d80080e413f2ed
[+] Thread Key found privkey 49dccfd96dc5df56487436f5a1b18c4f5d34f65ddb48cb5ee3579364de939b0c
[+] Publickey 04d4e6fa664bd75a508c0ff0ed6f2c52da2add7c3f954d9c346d24318dbd2ecfc6805511f46262e10a25f252fd525af1cbcc46016b6cd0a7705037364309198da1
All points were found

real    164m32.904s
user    973m8.387s
sys     0m26.803s
```

Amount of RAM used ~4.5 GB, time to solve the sixteen public keys in the range of 64 bits key-space: 164 min (~2.7 hrs) using 6 threads

if we run the same command with `-n 0x1000000000000000 -k 4 -t 6` it use ~18 GB or RAM and solve the same keys in 60 minutes

```
All points were found

real    59m50.533s
user    329m29.836s
sys     0m22.752s
```

There are several variations to play with the values `-n` and `-k` but there are some minimal values required, n can not be less than 1048576 (2^20)

To get optimal performance the k values need to be base 2^x values, this is 1,2,4,8,16,32 ... 

### Valid n and maximun k values for specific 

```
+------+----------------------+-------------+
| bits |  n in hexadecimal    | k max value |
+------+----------------------+-------------+
|   20 |             0x100000 | 1 (default) |
|   22 |             0x400000 | 2           |
|   24 |            0x1000000 | 4           |
|   26 |            0x4000000 | 8           |
|   28 |           0x10000000 | 16          |
|   30 |           0x40000000 | 32          |
|   32 |          0x100000000 | 64          |
|   34 |          0x400000000 | 128         |
|   36 |         0x1000000000 | 256         |
|   38 |         0x4000000000 | 512         |
|   40 |        0x10000000000 | 1024        |
|   42 |        0x40000000000 | 2048        |
|   44 |       0x100000000000 | 4096        |
|   46 |       0x400000000000 | 8192        |
|   48 |      0x1000000000000 | 16384       |
|   50 |      0x4000000000000 | 32768       |
|   52 |     0x10000000000000 | 65536       |
|   54 |     0x40000000000000 | 131072      |
|   56 |    0x100000000000000 | 262144      |
|   58 |    0x400000000000000 | 524288      |
|   60 |   0x1000000000000000 | 1048576     |
|   62 |   0x4000000000000000 | 2097152     |
|   64 |  0x10000000000000000 | 4194304     |
+------+----------------------+-------------+
```
 
**If you exceed the max value of K the program can have a unknow behavior, the program can have a suboptimal performance, or in the wrong cases you can missing some hits and have an incorrect SPEED.**

Note for user that want use it with SWAP memory. IT DOESN'T WORK with Swap Memory was made to small chucks of memory also is slowly.   

### What values use according to my current RAM:

2 G
-k 128

4 G
-k 256

8 GB
-k 512

16 GB
-k 1024

32 GB
-k 2048

64 GB
-n 0x100000000000 -k 4096

128 GB
-n 0x400000000000 -k 4096

256 GB
-n 0x400000000000 -k 8192

512 GB
-n 0x1000000000000 -k 8192

1 TB
-n 0x1000000000000 -k 16384

2 TB
-n 0x4000000000000 -k 16384

4 TB
-n 0x4000000000000 -k 32768

8 TB
-n 0x10000000000000 -k 32768


### Testing puzzle 63 bits

Publickey:

```
0365ec2994b8cc0a20d40dd69edfe55ca32a54bcbbaa6b0ddcff36049301a54579
```

Command
```
time ./keyhunt -m bsgs -t 8 -f tests/63.pub -k 512 -s 0 -S -b 63
```

output:

```
[+] Version 0.2.230430 Satoshi Quest, developed by AlbertoBSD
[+] Threads : 8
[+] K factor 512
[+] Turn off stats output
[+] Mode BSGS secuential
[+] Opening file tests/63.pub
[+] Added 1 points from file
[+] Bit Range 63
[+] -- from : 0x4000000000000000
[+] -- to   : 0x8000000000000000
[+] N = 0x100000000000
[+] Bloom filter for 2147483648 elements : 7361.33 MB
[+] Bloom filter for 67108864 elements : 230.04 MB
[+] Bloom filter for 2097152 elements : 7.19 MB
[+] Allocating 32.00 MB for 2097152 bP Points
[+] Reading bloom filter from file keyhunt_bsgs_4_2147483648.blm .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_6_67108864.blm .... Done!
[+] Reading bP Table from file keyhunt_bsgs_2_2097152.tbl .... Done!
[+] Reading bloom filter from file keyhunt_bsgs_7_2097152.blm .... Done!
[+] Thread Key found privkey 7cce5efdaccf6808
[+] Publickey 0365ec2994b8cc0a20d40dd69edfe55ca32a54bcbbaa6b0ddcff36049301a54579
All points were found00000000
[+] Thread 0x7cf4d00000000000
real    4m11.358s
user    26m23.474s
sys     0m20.061s

```

## Is my speed real?

Since this is still a beta version we can have some doubt about the speed showed in the bsgs mode.

To check this  i prepare a set of test publickeys to be found at some specific time according to your speed.

With  1 Petakeys/s the publickey will be found in 2 minutes:
Privatekey: 8000000000000001aa535d3d0c0000
Publickey : 02af4535880d694d660031a161c53a6889c45d2de513454858e94739f9c790768b

With 10 Petakeys/s the publickey will be found in 2 minutes:
Privatekey: 8000000000000010a741a462780000
Publickey : 025deee1657cd5d363cff23ec1b14781e504cbb6292c273e515d73f98065131d40

With 50 Petakeys/s the publickey will be found in 2 minutes:
Privatekey: 8000000000000053444835ec580000
Publickey : 03c13e9c6e5cbe2ac06817e4d8fd0a3e836f1a121aab91bb67ef44747b25c7d791

With  1 Exakey/s the publickey will be found in 2 minutes:
Privatekey: 800000000000068155a43676e00000
Publickey : 022b6a74badcc4c3d8fab7d01ddc1854b9d8f262172789b2aa1bb7fd42cc1b2817

With  5 Exakeys/s the publickey will be found in 2 minutes
Privatekey: 8000000000002086ac351052600000
Publickey : 024cf9e44f808e7b0bbb12a57ff63e3a8407cba1816f5e31e815d33d70e4a95a7f

With 10 Exakeys/s the publickey will be found in 2 minutes
Privatekey: 800000000000410d586a20a4c00000
Publickey : 02ee0cf78d13b4aae9c8777a0f93dff7f5be3855bd2c0f85370f861c69bb5b533a

Select one publickey that fit to your current speed save it in a file `testpublickey.txt` and test it with:

```
./keyhunt -m bsgs -f testpublickey.txt -b 120 -q
```

Change the values of k, n and t


The publickeys should be found in some 2 minutes after the load of the files

Change your n or k values according to your current memory and remember not exceed the k value of each N please check the table https://github.com/albertobsd/keyhunt#valid-n-and-k-values



This program was possible thanks to 
- IceLand
- kanhavishva
- XopMC
- WanderingPhilosopher
- Malboro Man
- NetSec
- Jean Luc Pons
- All the group of CryptoHunters that made this program possible
- All the users that tested it, report bugs, requested improvements and shared his knowledge.




