# uftrace-tutorial-for-LSM-tree

This document is written by [Min-guk Choi](https://github.com/korea-choi).

## [LevelDB](https://github.com/google/leveldb)
**1. Download LevelDB**  
```
git clone --recurse-submodules https://github.com/google/leveldb.git
```

**2. Add -pg option on [leveldb/CMakeList.txt](https://github.com/google/leveldb/blob/main/CMakeLists.txt) for uftrace**  
```diff
+ # Add -pg option for uftrace
+ set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  # Disable C++ exceptions.
```

**3. Build**
  ```
  mkdir -p build && cd build
  cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build .
  ```

**4. Record LevelDB db_bench**
  ```
  cd build
  uftrace record db_bench --benchmarks="fillseq" --num=10000
  ```
  - Find more LevelDB db_bench options [here](https://github.com/google/leveldb/blob/main/benchmarks/db_bench.cc).

**5. uftrace**  
**1) replay: function call tracing**  
  ```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 10037] | _GLOBAL__sub_I_leveldb::test::RandomString() {
  40.707 us [ 10037] |   std::ios_base::Init::Init();
   0.109 us [ 10037] |   __cxa_atexit();
  41.887 us [ 10037] | } /* _GLOBAL__sub_I_leveldb::test::RandomString */
            [ 10037] | _GLOBAL__sub_I_main() {
   0.050 us [ 10037] |   std::ios_base::Init::Init();
   0.043 us [ 10037] |   __cxa_atexit();
   0.330 us [ 10037] | } /* _GLOBAL__sub_I_main */
            [ 10037] | main() {
            [ 10037] |   leveldb::Options::Options() {
            [ 10037] |     leveldb::BytewiseComparator() {
   0.076 us [ 10037] |       __cxa_guard_acquire();
   0.048 us [ 10037] |       __cxa_guard_release();
   0.442 us [ 10037] |     } /* leveldb::BytewiseComparator */
            [ 10037] |     leveldb::Env::Default() {
   0.037 us [ 10037] |       __cxa_guard_acquire();
   0.032 us [ 10037] |       leveldb::Env::Env();
   0.111 us [ 10037] |       std::condition_variable::condition_variable();
   0.151 us [ 10037] |       operator new();
   0.060 us [ 10037] |       operator new();

  ```

**2) graph: function call graph**  
  ```
$ uftrace graph
# Function Call Graph for 'db_bench' (session: 93042af10e4a32a8)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
  263.921 ms : (1) db_bench
   42.397 us :  +-(1) _GLOBAL__sub_I_leveldb::test::RandomString
   41.196 us :  |  +-(1) std::ios_base::Init::Init
             :  |  | 
    0.100 us :  |  +-(1) __cxa_atexit
             :  | 
    0.304 us :  +-(1) _GLOBAL__sub_I_main
    0.047 us :  |  +-(1) std::ios_base::Init::Init
             :  |  | 
    0.035 us :  |  +-(1) __cxa_atexit
             :  | 
  158.081 ms :  +-(1) main
    2.678 us :  |  +-(4) leveldb::Options::Options
    0.555 us :  |  |  +-(4) leveldb::BytewiseComparator
    0.055 us :  |  |  |  +-(1) __cxa_guard_acquire
             :  |  |  |  | 
    0.046 us :  |  |  |  +-(1) __cxa_guard_release
             :  |  |  | 
    1.497 us :  |  |  +-(4) leveldb::Env::Default
    0.032 us :  |  |     +-(1) __cxa_guard_acquire
             :  |  |     | 
    0.027 us :  |  |     +-(1) leveldb::Env::Env
             :  |  |     | 
    0.115 us :  |  |     +-(1) std::condition_variable::condition_variable
             :  |  |     | 
    0.211 us :  |  |     +-(2) operator new
             :  |  |     | 
    0.278 us :  |  |     +-(1) getrlimit
  ```

**3) tui: terminal UI**  
  ```
$uftrace tui
    TOTAL TIME : FUNCTION
  263.921 ms : (1) db_bench
   42.397 us :  ├▶(1) _GLOBAL__sub_I_leveldb::test::RandomString
             :  │
    0.304 us :  ├▶(1) _GLOBAL__sub_I_main
             :  │
  158.081 ms :  ├▶(1) main
             :  │
  105.797 ms :  ├─(1) std::thread::_State_impl::_M_run
  105.796 ms :  │ (1) leveldb::Benchmark::ThreadBody
    0.301 us :  │  ├─(2) pthread_mutex_lock
             :  │  │
    3.410 us :  │  ├─(2) std::condition_variable::notify_all
             :  │  │
   76.570 us :  │  ├─(1) std::condition_variable::wait
             :  │  │
    0.443 us :  │  ├─(2) pthread_mutex_unlock
             :  │  │
    0.101 us :  │  ├─(1) leveldb::Histogram::Clear
             :  │  │
    0.561 us :  │  ├─(2) leveldb::_GLOBAL__N_1::PosixEnv::NowMicros
    0.145 us :  │  │ (2) gettimeofday
             :  │  │
  105.713 ms :  │  └─(1) leveldb::Benchmark::WriteSeq
  105.713 ms :  │    (1) leveldb::Benchmark::DoWrite
    7.309 ms :  │     ├─(10486) leveldb::test::CompressibleString
    1.021 ms :  │     │  ├─(20972) std::__cxx11::basic_string::resize
             :  │     │  │
  805.323 us :  │     │  ├─(20972) std::__cxx11::basic_string::_M_append
             :  │     │  │
  483.717 us :  │     │  └─(10486) operator delete

  ```

## [RocksDB](https://github.com/facebook/rocksdb)
**1. Download [RocksDB](https://github.com/facebook/rocksdb/blob/main/INSTALL.md) and Install dependencies**
* Dependencies
  - Install gflags : `sudo apt-get install libgflags-dev`
  - Install snappy : `sudo apt-get install libsnappy-dev`

* RocksDB
  - `git clone https://github.com/facebook/rocksdb.git`

**2. Add -pg option on [rocksdb/Makefile](https://github.com/facebook/rocksdb/blob/main/CMakeLists.txt) for uftrace**
  ```diff
  + # Add gcc -pg option for uftrace
  CLEAN_FILES = # deliberately empty, so we can append below.
  CFLAGS += ${EXTRA_CFLAGS}
  -- CXXFLAGS += ${EXTRA_CXXFLAGS}
  ++ CXXFLAGS += ${EXTRA_CXXFLAGS} -pg
  -- LDFLAGS += $(EXTRA_LDFLAGS) -pg
  ++ LDFLAGS += $(EXTRA_LDFLAGS) -pg
  MACHINE ?= $(shell uname -m)
  ARFLAGS = ${EXTRA_ARFLAGS} rs
  STRIPFLAGS = -S -x
  ```

**3. Build**
  - Make db_bench only: `make db_bench`
  - Make all: `make all`
  - Find RocksDB install details [here](https://github.com/facebook/rocksdb/wiki/Benchmarking-tools).  
  
**4. Record RocksDB db_bench**
  ```
  uftrace record db_bench
  ```
  - Find RocksDB db_bench options [here](https://github.com/facebook/rocksdb/blob/main/INSTALL.md).  

**5. uftrace**  
**1) replay : function call tracing**  
```
$ uftrace replay
# DURATION     TID     FUNCTION
            [ 14076] | _GLOBAL__sub_I_fLS::FLAGS_benchmarks::cxx11() {
            [ 14076] |   __static_initialization_and_destruction_0() {
   0.185 us [ 14076] |     std::ios_base::Init::Init();
   0.088 us [ 14076] |     __cxa_atexit();
   0.036 us [ 14076] |     __cxa_atexit();
            [ 14076] |     std::__cxx11::basic_string::basic_string() {
   2.617 us [ 14076] |       strlen();
            [ 14076] |       std::__cxx11::basic_string::_M_construct() {
   2.048 us [ 14076] |         std::__cxx11::basic_string::_M_create();
   0.154 us [ 14076] |         memcpy();
   2.558 us [ 14076] |       } /* std::__cxx11::basic_string::_M_construct */
   8.249 us [ 14076] |     } /* std::__cxx11::basic_string::basic_string */
            [ 14076] |     std::__cxx11::basic_string::_M_construct() {
   0.072 us [ 14076] |       std::__cxx11::basic_string::_M_create();
   0.057 us [ 14076] |       memcpy();
   0.337 us [ 14076] |     } /* std::__cxx11::basic_string::_M_construct */
   0.725 us [ 14076] |     google::FlagRegisterer::FlagRegisterer();
   0.042 us [ 14076] |     __cxa_atexit();
   2.743 us [ 14076] |     google::FlagRegisterer::FlagRegisterer();
   0.292 us [ 14076] |     google::FlagRegisterer::FlagRegisterer();
```

**2) graph: function call graph**  
```
$ uftrace graph
# Function Call Graph for 'db_bench' (session: d9c40b08019dd6c8)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
    2.857  s : (1) db_bench
    2.001 ms :  +-(1) _GLOBAL__sub_I_fLS::FLAGS_benchmarks::cxx11
    1.998 ms :  | (1) __static_initialization_and_destruction_0
    0.185 us :  |  +-(1) std::ios_base::Init::Init
             :  |  | 
    2.086 us :  |  +-(34) __cxa_atexit
             :  |  | 
   25.533 us :  |  +-(38) std::__cxx11::basic_string::basic_string
    9.717 us :  |  |  +-(38) strlen
             :  |  |  | 
    7.481 us :  |  |  +-(38) std::__cxx11::basic_string::_M_construct
    2.110 us :  |  |     +-(2) std::__cxx11::basic_string::_M_create
             :  |  |     | 
    1.095 us :  |  |     +-(23) memcpy
             :  |  | 
    4.000 us :  |  +-(37) std::__cxx11::basic_string::_M_construct
    0.114 us :  |  |  +-(2) std::__cxx11::basic_string::_M_create
             :  |  |  | 
    0.932 us :  |  |  +-(22) memcpy
             :  |  | 
  126.887 us :  |  +-(327) google::FlagRegisterer::FlagRegisterer
             :  |  | 
    1.168 ms :  |  +-(43) rocksdb::Options::Options
   78.629 us :  |  |  +-(43) rocksdb::DBOptions::DBOptions
   60.217 us :  |  |  |  +-(43) rocksdb::Env::Default
   11.774 us :  |  |  |  |  +-(43) rocksdb::ThreadLocalPtr::InitSingletons
    7.418 us :  |  |  |  |  | (43) rocksdb::ThreadLocalPtr::Instance
    1.065 us :  |  |  |  |  |  +-(1) __cxa_guard_acquire
```

**3) tui: terminal UI**  
```
$ uftrace tui
  TOTAL TIME : FUNCTION
    2.857  s : (1) db_bench
    2.001 ms :  ├─(1) _GLOBAL__sub_I_fLS::FLAGS_benchmarks::cxx11
    1.998 ms :  │ (1) __static_initialization_and_destruction_0
    0.185 us :  │  ├─(1) std::ios_base::Init::Init
              :  │  │
    2.086 us :  │  ├─(34) __cxa_atexit
              :  │  │
    25.533 us :  │  ├▶(38) std::__cxx11::basic_string::basic_string
              :  │  │
    4.000 us :  │  ├▶(37) std::__cxx11::basic_string::_M_construct
              :  │  │
  126.887 us :  │  ├─(327) google::FlagRegisterer::FlagRegisterer
              :  │  │
    1.168 ms :  │  ├▶(43) rocksdb::Options::Options                                                                                             
              :  │  │
  575.538 us :  │  ├─(43) rocksdb::ColumnFamilyOptions::~ColumnFamilyOptions
  564.492 us :  │  │  ├─(86) std::_Sp_counted_base::_M_release
  535.588 us :  │  │  │  ├─(86) std::_Sp_counted_ptr::_M_dispose
    9.796 us :  │  │  │  │  ├─(43) rocksdb::port::Mutex::~Mutex
    1.613 us :  │  │  │  │  │  ├─(43) pthread_mutex_destroy
              :  │  │  │  │  │  │
    1.216 us :  │  │  │  │  │  └─(43) rocksdb::port::PthreadCall
              :  │  │  │  │  │
  376.456 us :  │  │  │  │  ├─(43) std::_Sp_counted_ptr_inplace::_M_dispose
  373.697 us :  │  │  │  │  │ (43) rocksdb::lru_cache::LRUCache::~LRUCache
  172.501 us :  │  │  │  │  │  ├─(731) rocksdb::port::Mutex::~Mutex
    22.442 us :  │  │  │  │  │  │  ├─(731) pthread_mutex_destroy
              :  │  │  │  │  │  │  │
    36.418 us :  │  │  │  │  │  │  └─(731) rocksdb::port::PthreadCall

```
## LevelDB/RocksDB with [YCSB-cpp](https://github.com/ls4154/YCSB-cpp/)
Yahoo! Cloud Serving Benchmark([YCSB](https://github.com/brianfrankcooper/YCSB/wiki)) written in C++.
This is a fork of the [YCSB-C](https://github.com/basicthinker/YCSB-C) with the following changes.
 
 * Easy to bind with LevelDB, RocksDB and LMDB
 * Make Zipf distribution and data value more similar to the original YCSB
 * Status and latency reports during benchmark


**1. Download YCSB-cpp**
```
git clone https://github.com/ls4154/YCSB-cpp.git
```

**2. Add -pg option on [YCSB-cpp/Makefile](https://github.com/ls4154/YCSB-cpp/blob/master/Makefile) for uftrace**
```diff
+ # Add -pg option for uftrace
+ CXXFLAGS += -pg

ifeq ($(DEBUG_BUILD), 1)
	CXXFLAGS += -g
else
	CXXFLAGS += -O2
	CPPFLAGS += -DNDEBUG
endif
```

**3. Bind either LevelDB or RocksDB with YCSB-cpp**
* LevelDB
```make
EXTRA_CXXFLAGS ?= -I/example/leveldb/include
EXTRA_LDFLAGS ?= -L/example/example/leveldb/build -lsnappy
BIND_LEVELDB ?= 1
```
* RocksDB
```make
EXTRA_CXXFLAGS ?= -I/example/rocksdb/include
EXTRA_LDFLAGS ?= -L/example/rocksdb -ldl -lz -lsnappy -lzstd -lbz2 -llz4
BIND_ROCKSDB ?= 1
```

**4. Build**
```
make
```

**5. Record YCSB**
- Load and record YCSB workload A with LevelDB:
```
uftrace record ycsb -run -db leveldb -P workloads/workloada -P leveldb/leveldb.properties -s
```
- Load, run and record YCSB workload B with RocksDB:
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
$uftrace graph
# Function Call Graph for 'ycsb' (session: 7357e48f43b1e2e0)
========== FUNCTION CALL GRAPH ==========
# TOTAL TIME   FUNCTION
   30.960  s : (1) ycsb
   88.230 us :  +-(1) _GLOBAL__sub_I_ycsbc::LeveldbDB::db_
   87.991 us :  | (1) __static_initialization_and_destruction_0
   36.825 us :  |  +-(1) std::ios_base::Init::Init
             :  |  | 
    0.828 us :  |  +-(19) __cxa_atexit
             :  |  | 
    0.646 us :  |  +-(19) std::allocator::allocator
             :  |  | 
   31.079 us :  |  +-(19) std::__cxx11::basic_string::basic_string
    1.893 us :  |  |  +-(19) std::__cxx11::basic_string::_M_local_data
             :  |  |  | 
    0.698 us :  |  |  +-(19) std::__cxx11::basic_string::_Alloc_hider::_Alloc_hider
             :  |  |  | 
    2.485 us :  |  |  +-(19) std::char_traits::length
             :  |  |  | 
   17.844 us :  |  |  +-(19) std::__cxx11::basic_string::_M_construct
   16.514 us :  |  |    (19) std::__cxx11::basic_string::_M_construct_aux
   15.111 us :  |  |    (19) std::__cxx11::basic_string::_M_construct
    0.531 us :  |  |     +-(19) __gnu_cxx::__is_null_pointer
             :  |  |     | 
    3.526 us :  |  |     +-(19) std::distance
    0.543 us :  |  |     |  +-(19) std::__iterator_category
             :  |  |     |  | 
    0.549 us :  |  |     |  +-(19) std::__distance
             :  |  |     | 
    0.855 us :  |  |     +-(25) std::__cxx11::basic_string::_M_data
  ```

**3) tui: terminal UI**  
  ```
  $ uftrace tui
    TOTAL TIME : FUNCTION
   30.960  s : (1) ycsb
   88.230 us :  ├─(1) _GLOBAL__sub_I_ycsbc::LeveldbDB::db_ 
   87.991 us :  │▶(1) __static_initialization_and_destruction_0
             :  │
   13.402 us :  ├▶(1) _GLOBAL__sub_I_ycsbc::BasicDB::mutex_
             :  │
    0.371 us :  ├▶(1) _GLOBAL__sub_I_ycsbc::DBFactory::Registry::cxx11
             :  │
    0.998 us :  ├▶(1) _GLOBAL__sub_I_StatusThread
             :  │
   65.370 us :  ├▶(1) _GLOBAL__sub_I_ycsbc::kOperationString
             :  │
    0.217 us :  ├▶(1) _GLOBAL__sub_I_leveldb::EnvPosixTestHelper::SetReadOnlyFDLimit
             :  │
    8.642  s :  ├─(1) main
    1.036 us :  │  ├─(1) ycsbc::utils::Properties::Properties
    0.900 us :  │  │ (1) std::map::map
    0.772 us :  │  │ (1) std::_Rb_tree::_Rb_tree
    0.624 us :  │  │ (1) std::_Rb_tree::_Rb_tree_impl::_Rb_tree_impl
    0.249 us :  │  │  ├─(1) std::allocator::allocator
    0.027 us :  │  │  │ (1) __gnu_cxx::new_allocator::new_allocator
             :  │  │  │
    0.027 us :  │  │  ├─(1) std::_Rb_tree_key_compare::_Rb_tree_key_compare
             :  │  │  │
    0.114 us :  │  │  └─(1) std::_Rb_tree_header::_Rb_tree_header
    0.028 us :  │  │    (1) std::_Rb_tree_header::_M_reset
             :  │  │
  691.842 us :  │  ├▶(1) ParseCommandLine
  ```
