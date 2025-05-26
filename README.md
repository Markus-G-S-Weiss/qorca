# qorca

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![GitHub top language](https://img.shields.io/github/languages/top/Markus-G-S-Weiss/qorca.svg)]()

*Created by Markus G. S. Weiss on 2024-10-24.*
*Last updated: 2025-05-26*

qorca is a lightweight Python tool that streamlines the process of submitting ORCA quantum chemistry calculations to SLURM job schedulers with intelligent resource allocation and job management. It automatically detects and configures itself based on the cluster environment it's running in.

## Features

- Automatically detects and adjusts PAL values in ORCA input files.
- Intelligently maps CPU allocation between ORCA and SLURM.
- Smart memory management with adjustable `%maxcore` settings.
- Configurable scratch directory management.
- Support for multiple ORCA versions (5.0.4, 6.0.0, 6.0.1).
- Automatic cluster detection with cluster-specific configurations.
- Multi-cluster support with optimized defaults for each environment.
- Dry-run capability for testing job submission scripts.
- Comprehensive error checking and validation.
- Cross-version compatibility with different ORCA installations.

## Programs Included

qorca provides a single command-line executable with extensive configuration options:
- **Job Submission**: Generates and submits tailored SLURM scripts for ORCA jobs.
- **Input File Modification**: Automatically adjusts PAL and memory settings.
- **Resource Management**: Optimizes computational resource allocation.

## Installation Instructions

### Requirements

- Python 3.6 or later.
- Access to a SLURM-managed HPC cluster.
- ORCA quantum chemistry package installed on the cluster.

### Installation

1. Clone the repository:

   ```
   git clone https://github.com/Markus-G-S-Weiss/qorca.git
   ```

2. Make the script executable:

   ```
   chmod +x qorca/qorca
   ```

3. Add to your PATH (optional):

   ```
   # Add to ~/.bashrc or ~/.bash_profile
   export PATH=$PATH:/path/to/qorca
   ```

### Customizing Cluster and ORCA Configurations

Before using qorca, you may need to adjust the cluster configurations and ORCA installation paths in the script. Open the `qorca` file in a text editor and modify the `CLUSTER_CONFIGS` dictionary near the top of the file:

```python
CLUSTER_CONFIGS = {
    "g2": {
        "default_partition": "sterling",
        "mem_per_cpu": 4000,  # MB
        "default_time": "7-00:00:00",  # 7 days
        "scratch_base_dir": "/scratch/ganymede2/$USER",  # Base directory for scratch files
        "exclude_nodes": {
            "sterling": ["compute-2-09-05"],  # GPU node(s) to exclude for sterling partition
        },
        "orca_versions": {
            '5.0.4': {
                'module_loads': ['module load gnu9/9.4.0', 'module load openmpi4/4.1.1', 'module load orca/5.0.4'],
                'orca_executable': '/opt/ohpc/pub/apps/orca/5.0.4/orca',
            },
            '6.0.0': {
                'module_loads': ['module load gnu12/12.4.0', 'module load openmpi4/4.1.6'],
                'orca_executable': '/path/to/your/orca_6_0_0/orca',
            },
            '6.0.1': {
                'module_loads': ['module load gnu12/12.4.0', 'module load openmpi4/4.1.6'],
                'orca_executable': '/path/to/your/orca_6_0_1/orca',
            },
        }
    },
    # Add configurations for your other clusters if needed
}
```

For each cluster configuration, you can customize:
- Default partition to use
- Memory allocation per CPU
- Default walltime
- Base scratch directory path
- Nodes to exclude for specific partitions
- ORCA versions with their module loads and executable paths

### Cluster Detection

The script automatically detects which cluster it's running on through:

1. **Environment variable**: You can explicitly set the cluster by defining the `CLUSTER_NAME` environment variable:
   ```bash
   export CLUSTER_NAME=g2
   ```

2. **SLURM configuration**: If the environment variable is not set, qorca will try to detect the cluster name from SLURM's configuration using `scontrol show config`.

3. **Default fallback**: If neither method works, it falls back to a default cluster (g2).

You can also customize the mapping between SLURM's cluster names and your configuration keys by modifying the `cluster_mapping` dictionary in the `detect_cluster` function:

```python
cluster_mapping = {
    'g2': 'g2',
    'juno': 'juno',
    # Add your custom mappings here
}
```

## Usage

Run the executable with the following syntax:

   ```
   qorca [options] input_file
   ```

**Common Options:**

- `-c, --cpus NUM`: Number of CPUs to allocate in SLURM (default: matches PAL value)
- `-p, --pal NUM`: Set the PAL value in the ORCA input file
- `-m, --maxcore GB`: Set or overwrite the %maxcore value (in GB per core)
- `-t, --time TIME`: Wall time in SLURM format (default depends on cluster, e.g., 7 days for g2)
- `-q, --partition PART`: SLURM partition name (default depends on cluster, e.g., sterling for g2)
- `-v, --version VER`: ORCA version to use (default: 6.0.1)
- `-d, --dryrun`: Create the submission script without running it
- `--save-scratch`: Preserve scratch directory after job completion
- `-f, --save-all`: Copy all files from scratch directory back to working directory

## Example Usage

To submit a basic ORCA job:

   ```
   qorca calculation.inp
   ```

To specify resources and ORCA version:

   ```
   qorca calculation.inp -p 16 -t 2-00:00:00 -v 6.0.1
   ```

To create a submission script without submitting:

   ```
   qorca calculation.inp --dryrun
   ```

To adjust memory and preserve scratch:

   ```
   qorca calculation.inp -m 8 --save-scratch
   ```

## Advanced Options

- `-n, --name NAME`: Specify custom job name
- `-o, --output FILE`: Custom output file name
- `-e, --error FILE`: Custom error file name
- `-s, --scratch DIR`: Specify custom scratch directory path
- `-N, --node NODE`: Request specific compute node
- `-x, --exclude NODES`: Exclude specific nodes (comma-separated)
- `-F, --force`: Proceed despite resource mismatches
- `-k, --save-slurm`: Keep SLURM script after submission

## Scratch Directory Handling

The script handles scratch directories in the following way:

1. It tries to use the cluster-specific scratch directory defined in `CLUSTER_CONFIGS["cluster_name"]["scratch_base_dir"]`
2. If this directory doesn't exist or can't be created (e.g., due to permission issues):
   - An error message is displayed
   - The job is terminated with an exit code of 1
   
This approach ensures that jobs only run when a proper scratch directory is available, preventing potential issues with disk space in the working directory. Make sure that the scratch directory specified in the cluster configuration exists and is writable by the user.

## License

qorca is released under the GNU General Public License v3.0. See the [LICENSE](LICENSE) file for details.

## Authors

- **Markus G. S. Weiss**

  Primary developer and maintainer of qorca.

This project aims to simplify quantum chemistry workflow on HPC clusters and is continually improved with feedback from the computational chemistry community.

## Contact

For issues, suggestions, or contributions, please contact Markus G. S. Weiss or create issues or pull requests on the GitHub repository.
