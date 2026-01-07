# Cache Management Simulator

A multi-core L1 cache simulator implementing the MESI (Modified, Exclusive, Shared, Invalid) cache coherence protocol. This project simulates a 4-core system with L1 instruction caches and models cache hits, misses, evictions, and inter-core cache invalidations.

### Compilation

```bash
make
```

This will:
1. Create `obj/` and `bin/` directories if they don't exist
2. Compile all source files in `src/`
3. Link object files into executable: `bin/L1simulate`

### Clean Build

```bash
make clean
make
```

## Usage

### Basic Usage

```bash
./bin/L1simulate -t <tracefile_prefix> -s <s> -E <E> -b <b>
```

### Command-Line Options

- `-t <tracefile>`: **Required.** Prefix of trace files (e.g., `app1` loads `app1_proc0.trace`, `app1_proc1.trace`, etc.)
- `-s <s>`: **Required.** Number of set index bits (number of sets = 2^s)
- `-E <E>`: **Required.** Associativity (cache lines per set)
- `-b <b>`: **Required.** Number of block offset bits (block size = 2^b bytes)
- `-o <outfilename>`: Optional. Output file for logging results (useful for plotting/analysis)
- `-d`: Optional. Enable debug mode (prints cache state after each instruction)
- `-h`: Display help message

### Examples

Simulate a 16-set, 4-way associative cache with 64-byte blocks using `app1` traces:
```bash
./bin/L1simulate -t example_traces/app1 -s 4 -E 4 -b 6
```

Same configuration with debug output and file logging:
```bash
./bin/L1simulate -t example_traces/app1 -s 4 -E 4 -b 6 -d -o results.txt
```

Simulate a larger cache with 256 sets, 8-way associativity, 128-byte blocks:
```bash
./bin/L1simulate -t example_traces/app2 -s 8 -E 8 -b 7
```

## Trace File Format

Trace files contain one memory operation per line:
- Format: `[R|W] <hexadecimal_address>`
- `R` = Read operation
- `W` = Write operation
- Address in hexadecimal format

Example trace content:
```
R 0x7fff1234
W 0x7fff1238
R 0x7fff123c
R 0x80000000
```

## MESI Protocol

This simulator implements the MESI (Modified, Exclusive, Shared, Invalid) cache coherence protocol:

- **INVALID (I)**: Cache line contains no valid data
- **EXCLUSIVE (E)**: Cache line is valid, owned by this cache only, unmodified
- **SHARED (S)**: Cache line is valid, shared with other caches, unmodified
- **MODIFIED (M)**: Cache line is valid, owned by this cache only, modified

### State Transitions

When a cache processes a memory request:
1. **Cache Hit**: Line exists in valid state (E, S, or M)
2. **Cache Miss**: Line not present or in INVALID state; initiated by bus request
3. **Write**: May trigger bus traffic and invalidations in other caches
4. **Eviction**: LRU line replacement; if MODIFIED, triggers writeback

## Performance Metrics

The simulator tracks and reports:

- **Per-Core Metrics**:
  - Read/write counts
  - Hit/miss counts and rates
  - Evictions and writebacks
  - Bus invalidations
  - Data traffic (bytes transferred)

- **System-Wide Metrics**:
  - Total bus transactions
  - Total invalidations
  - Total bus traffic
  - Global cycle count

## Configuration Parameters

### Cache Parameters

The cache behavior is determined by three parameters:

1. **s (Set Index Bits)**: 
   - Number of sets = 2^s
   - Valid range: typically 4-12 (16 to 4,096 sets)

2. **E (Associativity)**:
   - Cache lines per set
   - Valid range: typically 1-16
   - E=1: Direct mapped
   - E=N: N-way set associative

3. **b (Block Bits)**:
   - Block size = 2^b bytes
   - Valid range: typically 4-10 (16 to 1,024 bytes)

### Example Cache Sizes

- s=4, E=4, b=6: 16 sets × 4 lines × 64 bytes = 4 KB cache
- s=6, E=8, b=6: 64 sets × 8 lines × 64 bytes = 32 KB cache
- s=8, E=8, b=7: 256 sets × 8 lines × 128 bytes = 256 KB cache

## Output

The simulator generates console output with:
1. Initialization messages
2. Per-core statistics (if debug mode enabled)
3. System-wide summary statistics
4. Cache miss rates and bus traffic analysis

If an output file is specified with `-o`, results are logged to that file for further analysis.

## Implementation Details

### LRU Replacement
The simulator uses a Last-Recently-Used (LRU) replacement policy. When a cache set is full and a miss occurs, the least recently used cache line is evicted.

### Bus Snooping
Caches monitor the shared bus for coherence-related transactions:
- Read/write requests from other cores
- Invalidation commands
- Cache-to-cache data transfers

### Cycle Accounting
The simulator tracks:
- Execution time (instruction cycles)
- Idle time (waiting for memory)
- Bus occupancy (for multi-core synchronization)

## Debug Mode

Enable with `-d` flag for detailed trace output showing:
- Each memory operation processed
- Cache hit/miss determination
- State transitions (INVALID → SHARED → MODIFIED, etc.)
- Bus transactions
- Cache evictions

Useful for understanding simulator behavior on small trace files.

## Building and Testing

### Quick Start
```bash
# Build the project
make

# Run with example traces
./bin/L1simulate -t example_traces/app1 -s 4 -E 4 -b 6

# Run with debug output
./bin/L1simulate -t example_traces/app1 -s 4 -E 4 -b 6 -d
```

### Clean Compiled Objects
```bash
make clean
```

## Notes

- The simulator processes 4 cores in parallel, reading from `app*_proc0.trace` through `app*_proc3.trace`
- All trace files for a given application must be present in the specified location
- Memory addresses are 32-bit unsigned integers
- Cache operations are processed cycle-by-cycle with bus contention modeling
- Statistics are reported both per-core and system-wide
