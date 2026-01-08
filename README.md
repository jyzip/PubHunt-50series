# PubHunt — GPU-Accelerated Public Key Hunting for Bitcoin Puzzles

![alt text](https://raw.githubusercontent.com/phrutis/LostCoins/main/Others/puzzle.jpg "PubHunt")
<br/>
Forked from https://github.com/kanhavishva/PubHunt
<br/>

## Overview

PubHunt is a high-performance, GPU-only tool designed to search for compressed Bitcoin public keys that match a given hash160.

This fork introduces significant internal changes and optimisations, with a strong focus on improved utilisation of NVIDIA RTX 50-series GPUs. The project is an ongoing optimisation effort, and further performance tuning and architectural improvements are expected.

The core objective remains unchanged: locate public keys corresponding to Bitcoin puzzle addresses where no public keys are publicly available.

---

## Purpose and Use Case

This software is only relevant for specific Bitcoin puzzle transactions, such as:
https://www.blockchain.com/btc/tx/08389f34c98c606322740c0be6a7125d9860bb8d5cb182c02f98461e5fa6cd15

Target puzzles typically include (but are not limited to):
64, 66, 67, 68, 69, 71, 72

For these puzzles:
- No public keys are known
- Only the address (hash160) is available
- If a valid public key is found, the Collider algorithm can then be used to attempt solving the puzzle

This tool does not attempt to recover private keys directly.

---

## How It Works

1. A random compressed public key is generated on the GPU
2. The public key is hashed to produce a hash160
3. The result is compared against one or more target puzzle hash160 values

Simplified flow:

Random Public Key  
→ SHA256  
→ RIPEMD160  
→ Compare with target hash160

If a match is found, the corresponding public key is written to the output file.

---

## Probability and Theory

In a 256-bit keyspace:

- One puzzle address (RIPEMD160) corresponds to  
  79,228,162,514,264,337,593,543,950,336 possible private keys
- Each private key maps to exactly one public key
- Therefore, the same number of public keys map to a single puzzle address

While the absolute odds remain extremely low, the probability of encountering a public key collision is significantly higher than guessing a specific private key directly. PubHunt exploits this property using massive parallelism on modern GPUs.

---

## GPU-Only Implementation

- No CPU search support
- CUDA-based implementation
- Optimised kernel scheduling and memory usage
- This fork includes improvements specifically targeting Ada-Next / Blackwell-class GPUs, including RTX 50-series cards
- Performance tuning is ongoing and may change between revisions

---

## Usage
```
PubHunt.exe -h
```
PubHunt [-check] [-h] [-v]  
        [-gi GPU ids: 0,1...]  
        [-gx gridsize: g0x,g0y,g1x,g1y,...]  
        [-o outputfile]  
        [inputFile]

-v                       : Print version information  
-gi gpuId1,gpuId2,...    : List of GPU(s) to use (default: 0)  
-gx g1x,g1y,g2x,g2y,...  : Specify GPU kernel grid size  
                           Default: 8 * (SM count), 128  
-o outputfile            : Write results to specified file  
-l                       : List CUDA-enabled devices  
-check                   : Validate CPU vs GPU kernel correctness  
inputFile                : Text file containing hash160 values  
                           One per line, hex encoded  

---

## Example Runs
```
PubHunt.exe -gx 231072,1024 puzzle64.txt
```
```  
PubHunt.exe -gx 231072,1024 hashes160.txt  
```


## Example Output
```
PubHunt v1.01 (Jan 2026 JYzip Edition)

DEVICE       : GPU  
GPU IDS      : 0  
GPU GRIDSIZE : 231072x1024  
NUM HASH160  : 1  
OUTPUT FILE  : Found.txt  
GPU          : GPU #0 NVIDIA GeForce RTX 5090 (170x0 cores) Grid(231072x1024)

[00:02:05] [GPU: 5191.80 Gkeys/s] [T: 650,081,652,965,376] [F: 0]

```


## Building

### Windows

Required:
- Microsoft Visual Studio Community 2022
- NVIDIA CUDA Toolkit 13.1

This fork assumes modern CUDA-capable hardware and is tested primarily against recent NVIDIA GPU architectures.

---

## Development Status

- Actively optimised fork
- Focused on better scheduling, throughput, and occupancy on RTX 50-series GPUs
- Not considered feature-complete
- Kernel behaviour and performance characteristics may change

---

## License

PubHunt is licensed under the GPLv3.
