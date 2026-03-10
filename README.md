fpt-max-cut
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![C++](https://img.shields.io/badge/C++-17-blue.svg)](https://isocpp.org/)
[![CMake](https://img.shields.io/badge/CMake-2.8+-064F8C.svg)](https://cmake.org/)
[![Linux](https://img.shields.io/badge/Linux-supported-success.svg)](https://github.com/KaHIP/fpt-max-cut)
[![macOS](https://img.shields.io/badge/macOS-supported-success.svg)](https://github.com/KaHIP/fpt-max-cut)
[![GitHub Stars](https://img.shields.io/github/stars/KaHIP/fpt-max-cut)](https://github.com/KaHIP/fpt-max-cut/stargazers)
[![GitHub Issues](https://img.shields.io/github/issues/KaHIP/fpt-max-cut)](https://github.com/KaHIP/fpt-max-cut/issues)
[![Last Commit](https://img.shields.io/github/last-commit/KaHIP/fpt-max-cut)](https://github.com/KaHIP/fpt-max-cut/commits)
[![ALENEX 2020](https://img.shields.io/badge/ALENEX'20-10.1137/1.9781611976007.3-blue)](https://doi.org/10.1137/1.9781611976007.3)
[![arXiv](https://img.shields.io/badge/arXiv-1905.10902-b31b1b.svg)](https://arxiv.org/abs/1905.10902)
[![Homebrew](https://img.shields.io/badge/homebrew-available-orange)](https://github.com/KaHIP/homebrew-kahip)
[![Heidelberg University](https://img.shields.io/badge/Heidelberg-University-c1002a)](https://www.uni-heidelberg.de)
=====

<p align="center">
  <img src="https://raw.githubusercontent.com/KaHIP/fpt-max-cut/master/logo/banner.png" alt="fpt-max-cut Logo" width="900"/>
</p>

**fpt-max-cut** implements FPT-based data reduction rules (kernelization) for the **maximum cut problem** on undirected graphs. Given a graph, the maximum cut problem asks for a partition of the vertices into two sets such that the total weight of edges between the sets is maximized. Part of the [KaHIP](https://github.com/KaHIP) organization.

| | |
|:--|:--|
| **What it solves** | Reduce graph size via kernelization while preserving the optimal maximum cut value |
| **Techniques** | FPT data reduction rules, kernelization, linear kernel computation |
| **Solvers** | MQLib (included), optional BiqMac and Localsolver integration |
| **Requires** | C++17 compiler (GCC 7+), CMake 2.8+, OpenMPI, CGAL, Sparsehash |

## Quick Start

### Install via Homebrew

```bash
brew install KaHIP/kahip/fpt-max-cut
```

### Build from source

```bash
git clone --recursive https://github.com/KaHIP/fpt-max-cut.git && cd fpt-max-cut
./build.sh
```

### Run

```bash
# Kernelization benchmark on a graph file
./build/fpt_max_cut -action kernelization -f example-graphs/example -iterations 1 -total-allowed-solver-time 10

# Generate random graphs and benchmark kernelization
./build/fpt_max_cut -action kernelization -sample-kagen 16 -num-nodes 64 -num-edges-lo 0 -num-edges-hi 256 -total-allowed-solver-time 1

# Linear kernel computation
./build/fpt_max_cut -action linear-kernel -f example-graphs/example -iterations 1
```

Or run the included example script:
```bash
./run-example.sh
```

---

## How It Works

The maximum cut problem is NP-hard. This project applies **fixed-parameter tractable (FPT) kernelization** to reduce the graph before solving. The data reduction rules provably preserve the optimal solution while shrinking the graph, often dramatically. The reduced graph is then solved using included or external solvers.

The pipeline:
1. **Load** a graph (multiple formats supported)
2. **Apply** data reduction rules (one-way and two-way reducers, signed reductions)
3. **Solve** the reduced graph using MQLib, BiqMac, or Localsolver
4. **Lift** the solution back to the original graph

---

## Executables

| Binary | Description |
|:-------|:------------|
| `benchmark` | Main benchmarking tool (release mode) |
| `benchmark-debug` | Debug version with assertions |
| `find-kernelization-general` | Explore general kernelization rules |
| `find-kernelization-weighted` | Explore weighted kernelization rules |
| `find-kernelization-grid` | Grid-based kernelization search |
| `double-clique-solver` | Clique-based solver |
| `unweight-an-instance` | Remove edge weights from a graph |
| `remove-double-edges` | Clean up duplicate edges |

---

## Command Line Usage

```
./build/fpt_max_cut [options]
```

### Core Options

| Option | Description | Default |
|:-------|:-----------|:--------|
| `-action <name>` | Action: `kernelization` or `linear-kernel` | |
| `-f <file>` | Input graph file | |
| `-seed <int>` | Random seed | |
| `-iterations <int>` | Number of repetitions | `1` |
| `-total-allowed-solver-time <sec>` | Time limit in seconds (`-1` to disable solvers) | auto |
| `-benchmark-output <file>` | Output file for benchmark data | |

### Kernelization Options

| Option | Description |
|:-------|:-----------|
| `-do-signed-reduction` | Enable signed reductions |
| `-do-mc-extension-algo` | Compute clique forest and run max-cut extension |
| `-support-weighted-result` | Allow weighted output graphs (better reduction) |
| `-output-graphs-dir <path>` | Save kernelized and original graphs |

### Graph Generation Options

| Option | Description | Default |
|:-------|:-----------|:--------|
| `-sample-kagen <int>` | Generate random graphs per KaGen type (BA, GNM, RGG2D, RGG3D, RHG) | |
| `-num-nodes <int>` | Number of vertices | `8192` |
| `-num-edges-lo <int>` | Lower bound on edges | |
| `-num-edges-hi <int>` | Upper bound on edges | |

### Solver Options

| Option | Description |
|:-------|:-----------|
| `-no-biqmac` | Disable BiqMac solver |
| `-no-localsolver` | Disable Localsolver |
| `-locsearch-iterations <int>` | Number of local search iterations |

---

## Graph Formats

Three input formats are supported:

**Default format:**
```
#nodes #edges
x_1 y_1
...
x_m y_m
```

**Edge list** (`.edges` suffix):
```
x_1 y_1
...
x_m y_m
```
Node and edge counts are computed automatically.

**Adjacency list** (`.graph` suffix, METIS-like):
```
#nodes #edges is_weighted
neighbors_of_node_1 [weights]
neighbors_of_node_2 [weights]
...
```

---

## Building from Source

### Requirements

- C++17 compiler (GCC 7+)
- CMake 2.8+
- OpenMPI (`libopenmpi-dev`)
- CGAL (`libcgal-dev`)
- Google Sparsehash (`libsparsehash-dev`)

### Install dependencies (Ubuntu/Debian)

```bash
sudo apt-get install gcc g++ libopenmpi-dev libcgal-dev libsparsehash-dev
```

### Build

```bash
git clone --recursive https://github.com/KaHIP/fpt-max-cut.git && cd fpt-max-cut
./build.sh
```

Or manually:
```bash
git submodule update --init --recursive
cd solvers/MQLib && make && cd ../..
mkdir build && cd build && cmake .. && make benchmark
```

### Optional Solvers

[BiqMac](http://biqmac.aau.at/) and [Localsolver](https://www.localsolver.com/) can be linked by specifying their binary paths in `build-config.json` before running cmake.

---

## Related Projects

| Project | Description |
|:--------|:------------|
| [KaHIP](https://github.com/KaHIP/KaHIP) | Karlsruhe High Quality Graph Partitioning |
| [VieCut](https://github.com/KaHIP/VieCut) | Shared-memory parallel minimum cut algorithms |
| [MQLib](https://github.com/MQLib/MQLib) | Max-Cut heuristic library (included as submodule) |

---

## Licence

fpt-max-cut is free software provided under the MIT License.
If you publish results using our algorithms, please cite:

```bibtex
@inproceedings{DBLP:conf/alenex/FerizovicHLM0S20,
  author    = {Damir Ferizovic and Demian Hespe and Sebastian Lamm and Matthias Mnich and Christian Schulz and Darren Strash},
  title     = {Engineering Kernelization for Maximum Cut},
  booktitle = {Proceedings of the 22nd Symposium on Algorithm Engineering and Experiments ({ALENEX} 2020)},
  pages     = {27--41},
  publisher = {{SIAM}},
  year      = {2020},
  doi       = {10.1137/1.9781611976007.3}
}
```
