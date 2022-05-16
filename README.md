# uftrace-tutorial-for-LSM-tree

This document is written by [Min-guk Choi](https://github.com/korea-choi).

## [LevelDB](https://github.com/google/leveldb)
**1. Download LevelDB**  
```
git clone --recurse-submodules https://github.com/google/leveldb.git
```

**2. Add -pg option on [leveldb/CMakeList.txt](https://github.com/google/leveldb/blob/main/CMakeLists.txt) for uftrace**  
``` cmake
# Add gcc -pg option for uftrace
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")
```

**3. Build**
  ```
  mkdir -p build && cd build
  cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build .
  ```

**4. Record LevelDB db_bench**
  ```
  cd build
  uftrace record db_bench
  ```
  - find LevelDB db_bench options. [here](https://github.com/google/leveldb/blob/main/benchmarks/db_bench.cc)

**5. uftrace**  
**1) replay: function call tracing**  
  ```
  uftrace replay
  ```

**2) graph: function call graph**  
  ```
  uftrace graph
  ```

**3) tui: terminal UI**  
  ```
  uftrace tui
  ```
**6. Filter**  
  - Below filiters are recommanded to find important LevelDB function only.  

|filter|description|
|---|---|
|--no-libcall|Do not show library calls.|
|-N ^leveldb::Slice::||
|-N ^leveldb::port::Mutex::|Do not show details of Mutex|
|-H ^std::|Hide std function |
|-H ^__gnu_cxx|Hide __gnu_cxx function|
|-H ^operator|Hide operators like new() and delete()|
|-H ^leveldb::Status|Hide [Status](https://github.com/google/leveldb/blob/main/include/leveldb/status.h) function that encapsulates the result of an operation.|
|-H ^leveldb::GetVarint|Hide [GetVarint](https://github.com/google/leveldb/blob/main/util/coding.h) function routines parse a value from the beginning of a Slice.|
|-H ^leveldb::PutVarint|Hide [PutVarint](https://github.com/google/leveldb/blob/main/util/coding.h) function routines append to a string.|
|-H ^leveldb::DecodeFixed|Hide [Decode](https://github.com/google/leveldb/blob/main/util/coding.h) function that read directly from a character buffer.|
|-H ^leveldb::EncodeFixed|Hide [Encode](https://github.com/google/leveldb/blob/main/util/coding.h) function that write directly from a character buffer.|
|-H ^leveldb::operator||
  
  - Try replay/graph with filters
  ```
  uftrace replay --no-libcall -N ^leveldb::Slice::
  ```

  - find more about uftrace filiter, [here](https://github.com/namhyung/uftrace/wiki/Filters).  

## [RocksDB](https://github.com/facebook/rocksdb)
**1. Download [RocksDB](https://github.com/facebook/rocksdb/blob/main/INSTALL.md) and Install dependencies**
* Dependencies
  - Install gflags : `sudo apt-get install libgflags-dev`
  - Install snappy : `sudo apt-get install libsnappy-dev`

* RocksDB
  - `git clone https://github.com/facebook/rocksdb.git`

**2. Add -pg option on [rocksdb/Makefile](https://github.com/facebook/rocksdb/blob/main/CMakeLists.txt) for uftrace**
  ``` Makefile
  # Add gcc -pg option for uftrace
  CXXFLAGS += ${EXTRA_CXXFLAGS} -pg
  LDFLAGS += $(EXTRA_LDFLAGS) -pg
  ```

**3. Build**
  - Make db_bench only: `make db_bench`
  - Make all: `make all`
  - find RocksDB install details, [here](https://github.com/facebook/rocksdb/wiki/Benchmarking-tools).  
  
**4. Record RocksDB db_bench**
  ```
  uftrace record db_bench
  ```
  - find RocksDB db_bench options. [here](https://github.com/facebook/rocksdb/blob/main/INSTALL.md)  

**5. uftrace**  
**1) replay : Function call Tracing**  
```
uftrace replay
```

**2) graph: Function call graph**  
```
uftrace graph
```

**3) tui: Function call graph**  
```
uftrace graph
```
## LevelDB/RocksDB with [YCSB-cpp](https://github.com/ls4154/YCSB-cpp/)
Yahoo! Cloud Serving Benchmark([YCSB](https://github.com/brianfrankcooper/YCSB/wiki)) written in C++.
This is a fork of [YCSB-C](https://github.com/basicthinker/YCSB-C) with following changes.
 
 * Easy to bind with LevelDB, RocksDB, LMDB
 * Make Zipf distribution and data value more similar to the original YCSB
 * Status and latency report during benchmark


**1. Download YCSB-cpp**
```
git clone https://github.com/ls4154/YCSB-cpp.git
```

**2. Add -pg option on [YCSB-cpp/Makefile](https://github.com/ls4154/YCSB-cpp/blob/master/Makefile) for uftrace**
```Make
CXXFLAGS += -pg
```

**3. Bind LevelDB/RocksDB (Choose One) with YCSB-cpp**
* LevelDB
```Make
EXTRA_CXXFLAGS ?= -I/example/leveldb/include
EXTRA_LDFLAGS ?= -L/example/example/leveldb/build -lsnappy
BIND_LEVELDB ?= 1
```
* RocksDB
```Make
EXTRA_CXXFLAGS ?= -I/example/rocksdb/include
EXTRA_LDFLAGS ?= -L/example/rocksdb -ldl -lz -lsnappy -lzstd -lbz2 -llz4
BIND_ROCKSDB ?= 1
```

**4. Build**
```
make
```

**5. Record ycsb**
- load and record ycsb workload A with leveldb:
```
uftrace record ycsb -run -db leveldb -P workloads/workloada -P leveldb/leveldb.properties -s
```
- load and run and record ycsb workload B with rocksdb:
```
uftrace record ycsb -load -run -db rocksdb -P workloads/workloadb -P rocksdb/rocksdb.properties -s
```

**6. uftrace**  
**1) replay: function call tracing**  
  ```
  uftrace replay
  ```

**2) graph: function call graph**  
  ```
  uftrace graph
  ```

**3) tui: terminal UI**  
  ```
  uftrace tui
  ```


