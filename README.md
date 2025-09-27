# qorca

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![GitHub top language](https://img.shields.io/github/languages/top/Markus-G-S-Weiss/qorca.svg)]()

*Created by Markus G. S. Weiss on 2024-10-24.*
*Last updated: 2025-09-27*

A Python tool for submitting ORCA quantum chemistry calculations to SLURM schedulers with flexible resource allocation and automatic cluster detection.

## Features

- Automatic PAL value detection and adjustment
- Flexible resource allocation (tasks, CPUs per task, memory per CPU)
- Memory usage warnings (75% threshold)
- Multi-cluster support with automatic detection
- Support for multiple ORCA versions (5.0.4, 6.0.0, 6.0.1)
- Scratch directory management
- Dry-run capability

## Installation

```bash
git clone https://github.com/Markus-G-S-Weiss/qorca.git
chmod +x qorca/qorca
```

### Configuration

Edit the `CLUSTER_CONFIGS` dictionary in the script to match your cluster setup:

```python
CLUSTER_CONFIGS = {
    "g2": {
        "default_partition": "sterling",      # Default SLURM partition
        "mem_per_cpu": 4000,                 # Default memory per CPU (MB)
        "default_time": "7-00:00:00",        # Default walltime
        "scratch_base_dir": "/scratch/ganymede2/$USER",  # Scratch directory base
        "exclude_nodes": {
            "sterling": ["g-07-02"],         # Nodes to exclude per partition
        },
        "orca_versions": {
            '6.0.1': {
                'module_loads': ['module load gnu12/12.4.0', 'module load openmpi4/4.1.6'],
                'orca_executable': '/path/to/your/orca',  # Path to ORCA binary
            },
        }
    }
}
```

For each cluster, customize:
- **Partition and timing**: Default partition name and walltime limits
- **Memory allocation**: Default memory per CPU for resource calculations
- **Scratch handling**: Base directory where temporary files are stored
- **Node management**: Exclude problematic nodes (e.g., GPU nodes for CPU jobs)
- **ORCA versions**: Available versions with required modules and executable paths

### Cluster Detection

The script automatically detects which cluster it's running on through a 3-step process:

1. **Environment variable**: Set explicitly with `export CLUSTER_NAME=g2`
2. **SLURM config**: Auto-detects from `scontrol show config` output
3. **Default fallback**: Uses g2 configuration if detection fails

Customize the cluster name mapping in the `detect_cluster` function:

```python
cluster_mapping = {
    'g2': 'g2',           # Maps SLURM cluster name to config key
    'juno': 'juno',
    'your_cluster': 'g2', # Map your cluster name to existing config
}
```

This mapping connects your SLURM cluster names to the configuration keys in `CLUSTER_CONFIGS`. If your cluster has a different name in SLURM, you can map it to an appropriate configuration.

## Usage

```
qorca [options] input_file
```

### Options

- `--tasks NUM`: Number of parallel tasks (default: matches PAL value)
- `-c, --cpus NUM`: CPUs per task (default: 1)
- `-M, --mem-per-cpu MB`: Memory per CPU in MB
- `-p, --pal NUM`: Set PAL value in ORCA input
- `-m, --maxcore GB`: Set %maxcore value (GB per core)
- `-t, --time TIME`: Wall time
- `-q, --partition PART`: SLURM partition
- `-v, --version VER`: ORCA version (default: 6.0.1)
- `-d, --dryrun`: Create script without submitting
- `--save-scratch`: Preserve scratch directory

**Less Common Options**
- `-n, --name NAME`: Custom job name
- `-o, --output FILE`: Custom output file
- `-e, --error FILE`: Custom error file
- `-s, --scratch DIR`: Custom scratch directory
- `-N, --node NODE`: Specific compute node
- `-x, --exclude NODES`: Exclude nodes (comma-separated)
- `-F, --force`: Proceed despite mismatches
- `-k, --save-slurm`: Keep SLURM script
- `-f, --save-all`: Copy all scratch files back

### Examples

```bash
# Basic submission
qorca calculation.inp

# Specify 16 parallel tasks with 2-day walltime
qorca calculation.inp --pal 16 -t 2-00:00:00

# 8 tasks with 2 CPUs each (16 total CPUs)
qorca calculation.inp --pal 8 --cpus 2

# Custom memory allocation
qorca calculation.inp --pal 4 --mem-per-cpu 8000

# Test script generation without submitting
qorca calculation.inp --dryrun

# High-memory calculation with scratch preservation
qorca calculation.inp --maxcore 12 --mem-per-cpu 16000 --save-scratch
```

**Note:** Total CPUs = Tasks × CPUs per Task

## License

qorca is released under the GNU General Public License v3.0. See the [LICENSE](LICENSE) file for details.

## Authors

- **Markus G. S. Weiss**

  Primary developer and maintainer of qorca.

This project aims to simplify quantum chemistry workflow on HPC clusters and is continually improved with feedback from the computational chemistry community.

## Contact

For issues, suggestions, or contributions, please contact Markus G. S. Weiss or create issues or pull requests on the GitHub repository.
