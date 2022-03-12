# sqlite3_bench-trace-slicing

This repo uses:  
 - [chalupa/dg](https://github.com/mchalupa/dg)
 - [ukontainer/sqlite-bench](https://github.com/ukontainer/sqlite-bench)

## 1. Compile

The `-rdynamic` (for printing backtraces) linking option is already been added to `MakeFile`  
```bash
make CC=/usr/bin/clang-9
```

## 2. Run
```
./sqlite-bench --benchmarks=fillrandsync
```
Then, the backtrace file `sqlite3_log_hhc.txt` will be in `/home/timhe` (can change in `sqlite3.c`) like:
```
---- 4540 ----
	./sqlite-bench(full_fsync+0x2f) [0x41f42f]
	./sqlite-bench(unixSync+0x13) [0x41f503]
	./sqlite-bench(sqlite3WalFrames+0xba4) [0x4254e4]
	./sqlite-bench(pagerWalFrames+0xb3) [0x4248f3]
	./sqlite-bench(sqlite3PagerCommitPhaseOne+0x165) [0x429975]
	./sqlite-bench(sqlite3BtreeCommitPhaseOne+0x68) [0x431328]
	./sqlite-bench(vdbeCommit+0x267) [0x441a37]
	./sqlite-bench(sqlite3VdbeHalt+0x781) [0x442bb1]
	./sqlite-bench(sqlite3VdbeExec+0x921e) [0x45171e]
	./sqlite-bench(sqlite3_step+0x1a0) [0x445a60]
	./sqlite-bench(benchmark_write+0x23a) [0x412c6a]
	./sqlite-bench(benchmark_run+0x696) [0x412626]
	./sqlite-bench(main+0x33f) [0x4141ff]
	/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7f4b0a7d2c87]
	./sqlite-bench(_start+0x2a) [0x411bfa]
  
  ---- 4540 ----
	./sqlite-bench(full_fsync+0x2f) [0x41f42f]
	./sqlite-bench(unixSync+0x13) [0x41f503]
	./sqlite-bench(walCheckpoint+0xd66) [0x42c4b6]
	./sqlite-bench(sqlite3WalCheckpoint+0x151) [0x42af31]
	./sqlite-bench(sqlite3BtreeCheckpoint+0xbf) [0x43af1f]
	./sqlite-bench(sqlite3_wal_checkpoint_v2+0x217) [0x4c1927]
	./sqlite-bench(sqlite3WalDefaultHook+0x2e) [0x4a2afe]
	./sqlite-bench(sqlite3_step+0x2b7) [0x445b77]
	./sqlite-bench(benchmark_write+0x23a) [0x412c6a]
	./sqlite-bench(benchmark_run+0x696) [0x412626]
	./sqlite-bench(main+0x33f) [0x4141ff]
	/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xe7) [0x7f4b0a7d2c87]
	./sqlite-bench(_start+0x2a) [0x411bfa]
```

## 3. Slicing

Generate `.bc` file
```bash
/usr/bin/clang-9 -Wall -I. -g -O2 -DNDEBUG -fno-discard-value-names -std=c99 -emit-llvm -c -o sqlite3.bc sqlite3.c
```
View the `.bc` file
```bash
llvm-dis-9 -o sqlite3.ll sqlite3.bc

vim sqlite3.ll
```
Slice the code
```bash
llvm-slicer -cutoff-diverging=false -sc 'fsync' -entry=sqlite3_step sqlite3.bc
```
