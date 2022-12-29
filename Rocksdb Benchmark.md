# Rocksdb Benchmark.sh

Runnining

```bash
./benchmark.sh
```

### Important Environment Variable

```bash
DB_DIR         // Path to write the database data directory
WAL_DIR        // Path to write the database WAL directory
OUTPUT_DIR     // Path of benchmark result (Default : /tmp)
NUM_KEYS       // The number of keys to use in the benchmark
VALUE_SIZE     // The size of the value to use in benchmark (Default : 400bytes)
KEY_SIZE       // This size of the keys to use in benchmark (Default : 20 bytes)
BlOCK_SIZE     // The size of database blocks used in benchmark(Default : 8kb)
NUM_THREADS    // The number of threads to use (default: 64)
CACHE_SIZE     // Size of the block cache (default: 16GB)
COMPRESSION_TYPE  // Default compression(default: zstd)
COMPRESSION_SIZE_PERCENT  // Value for compression_size_percent
MIN_LEVEL_TO_COMPRESS  // Value for min_level_to_compress for Leveled
DURATION            // Number of seconds for which the test runs
WRITES              // Number of writes for which the test runs
WRITE_BUFFER_SIZE_MB    // The size of the write buffer in MB (default: 128)
MAX_BACKGROUND_JOBS     // The value for max_background_jobs (default: 16)
REPORT_INTERVAL_SECONDS  // Value for report_interval_seconds
STATS_INTERVAL_SECONDS   // Value for stats_interval_seconds
```

### Benchmark testing methods

```c
bulkload         // Measures performance inserting in random order to database.
readrandom       // Measures performance reading existing keys randomly
multireadrandom  // Measures performance multiget method reading keys randomly
fwdrange         // Measure performance to randomly iterate over keys
fillseq_disable_wal  // Sequentially fill the database with no WAL
fillseq_enable_wal    // Sequentially fill the database with WAL
revrange         // Measure performance to randomly iterate rev order over keys
overwrite        // Measure performance to randomly overwrite keys into the database
updaterandom     // Measure performance to randomly update keys into database
readwhilewriting // Measure performance with one writer and multiple reader threads.
readwhilemerging  // Measure performance with multiple read and one merging threads
fwdrangewhilewriting // Measure performance with one writer and multiple iterator threads. 
revrangewhilewriting // Measure performance with one writer and multiple iterator threads. 

```

Example for testing

```bash
DB_DIR=/tmp/ WAL_DIR=/tmp/ NUM_THREADS=64  NUM_KEYS=900000000 CACHE_SIZE=6442450944 DURATION=5400 BLOCK_SIZE=160000 MB_WRITE_PER_SEC=2 tools/benchmark.sh readrandom
```




