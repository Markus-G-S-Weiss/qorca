# qorca

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![GitHub top language](https://img.shields.io/github/languages/top/Markus-G-S-Weiss/qorca.svg)]()

*Created by Markus G. S. Weiss on 2024-10-24.*

qorca is a lightweight Python tool that streamlines the process of submitting ORCA quantum chemistry calculations to SLURM job schedulers with intelligent resource allocation and job management.

## Features

- Automatically detects and adjusts PAL values in ORCA input files.
- Intelligently maps CPU allocation between ORCA and SLURM.
- Smart memory management with adjustable `%maxcore` settings.
- Configurable scratch directory management.
- Support for multiple ORCA versions (5.0.4, 6.0.0, 6.0.1).
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

### Customizing ORCA Installation Paths

Before using qorca, you need to adjust the paths to your ORCA installation in the script. Open the `qorca` file in a text editor and modify the `ORCA_VERSIONS` dictionary near the top of the file:

```python
ORCA_VERSIONS = {
    '5.0.4': {
        'module_loads': ['module load gnu9/9.4.0', 'module load openmpi4/4.1.1', 'module load orca/5.0.4'],
        'orca_executable': '/opt/ohpc/pub/apps/orca/5.0.4/orca',
    },
    '6.0.0': {
        'module_loads': ['module load gnu12/12.3.0', 'module load openmpi4/4.1.6'],
        'orca_executable': '/path/to/your/orca_6_0_0/orca',
    },
    '6.0.1': {
        'module_loads': ['module load gnu12/12.3.0', 'module load openmpi4/4.1.6'],
        'orca_executable': '/path/to/your/orca_6_0_1/orca',
    },
}
```

Adjust both the `module_loads` list and the `orca_executable` path for each ORCA version to match your cluster's configuration. You can also add additional ORCA versions as needed.

## Usage

Run the executable with the following syntax:

   ```
   qorca [options] input_file
   ```

**Common Options:**

- `-c, --cpus NUM`: Number of CPUs to allocate in SLURM (default: matches PAL value)
- `-p, --pal NUM`: Set the PAL value in the ORCA input file
- `-m, --maxcore GB`: Set or overwrite the %maxcore value (in GB per core)
- `-t, --time TIME`: Wall time in SLURM format (default: 1-00:00:00)
- `-q, --partition PART`: SLURM partition name (default: sterling)
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
   qorca calculation.inp -c 16 -p 16 -m 4 -t 2-00:00:00 -v 6.0.1
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

## License

qorca is released under the GNU General Public License v3.0. See the [LICENSE](LICENSE) file for details.

## Authors

- **Markus G. S. Weiss**

  Primary developer and maintainer of qorca.

This project aims to simplify quantum chemistry workflow on HPC clusters and is continually improved with feedback from the computational chemistry community.

## Contact

For issues, suggestions, or contributions, please contact Markus G. S. Weiss or create issues or pull requests on the GitHub repository.
