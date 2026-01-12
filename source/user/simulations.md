# Running Simulations

Simulations can be directly run in a python script or using the *command line interface* (CLI).

## Python Script

To run a simulation from Python, you need to provide a [configuration file](./config-file.md). Then you can start the simulation as follows:

```python
from illuminator.engine import Simulation

simulation = Simulation('<path/to/scenario_config.yaml>')
simulation.run()

```

## Command Line

You can use the command `scenario run` to start a simulation from the terminal:

```shell
illuminator scenario run <path/to/scenario_config.yaml>
```

*Feature added in October 2025 as part of the support from the Digital Competence Centre.*

# Running Simulations in Parallel

The Illuminator allows users to run multiple scenarios in parallel by distributing simulations across processors, reducing the overall runtime when executing many independent simulations.

The parallel execution feature distributes simulation workloads across multiple CPU cores or HPC compute nodes using the Message Passing Interface (MPI) via the mpi4py Python library. Each simulation is executed independently, making the feature ideal for exploring large parameter spaces as part of a sensitivity analysis workflow or batch scenario evaluations.

## Overview

The feature introduces two complementary functions in the `illuminator.parallel_scenarios` module:

| Function | Description |
|----------|-------------|
| `run_parallel(simlist, create_scenario_files=False)` | Runs a list of pre-configured `Simulation` objects in parallel. Optionally writes a scenario YAML file for each simulation. |
| `run_parallel_file(scenario_file)` | Expands a multi-parameter scenario YAML file into individual simulations and runs them in parallel. Generates scenario YAML files and a lookup table summarizing parameter combinations. Can be accessed from the CLI app via `illuminator scenario run_parallel <myscenario.yaml>`|

Both functions internally manage MPI rank assignment and ensure that each process runs a unique subset of the total simulations. If there are more simulations than MPI processes, simulations are distributed evenly across ranks. If fewer simulations exist, extra ranks remain idle.

## Usage

### 1. Using Predefined Simulations

If you already have a list of `Simulation` objects:

```python
#run_my_sims.py

from illuminator.engine import Simulation
from illuminator.parallel_scenarions import run_parallel

# Define a list of simulation objects
simlist = [
    Simulation("scenario1.yaml"),
    Simulation("scenario2.yaml"),
    Simulation("scenario3.yaml"),
]

# Run all simulations in parallel (using MPI)
run_parallel(simlist, create_scenario_files=True)
```

To launch in parallel with 4 processors, use `mpirun` or your cluster’s job scheduler:
```bash
mpirun -np 4 python run_my_sims.py
```

### 2. Using a Multi-Parameter Scenario File

The Illuminator also supports concurrently running multiple model parameter combinations from a single YAML scenario file. This approach has the upside of performing many simulations without manually creating separate scenario files. 

The function `run_parallel_file(scenario_file)`:
- Expands all parameter/state combinations automatically
- Writes derived scenario YAMLs to disk (e.g., myscenario_1.yaml, myscenario_2.yaml, ...)
- Executes simulations in parallel; each processor (MPI rank) runs a unique subset of simulations
- Generates a lookup table (scenariotable.csv) summarizing the parameter combinations

To run the simulations defined in a multi-parameter scenario file with 8 processors use `mpirun` or your cluster’s job scheduler:
```bash
mpirun -np 8 illuminator scenario run_parallel myscenario.yaml
```

#### YAML Schema for Multi-Parameters
To define multiple parameter values in a single scenario, use a `multi_parameters` section under each model. Each entry in multi_parameters specifies either a `list` or a `range` of values to iterate over. If you use a range, it must define integer values for the minimum, maximum, and step. Lists can include floating-point numbers. You can vary as many parameters across as many models as you like.

An optional `align_parameters` key under the scenario section specifies how the mulit_parameters should be combined. When set to `True`, the corresponding parameter lists or ranges are combined by index (aligned). This option requires all lists or ranges to be of equal length. When set to `False`, all possible parameter combinations are generated (Cartesian product). If align_parameters is not defined, it is set to False by default.

For example, depending on the `align_parameters` setting, the following example would produce:
- True → Generates 2 scenarios: (1000, 80) and (1500, 90)
- False → Generates 4 scenarios: (1000, 80), (1000, 90), (1500, 80) and (1500, 90)

```yaml
scenario:
  name: "Neighborhood_scenario1"
  start_time: '2007-07-02 00:00:00'
  end_time: '2007-07-02 23:45:00'
  time_resolution: 900
  align_parameters: True

models:
  - name: Battery1
    type: Battery
    parameters:
      max_p: 500
      min_p: -500
      discharge_efficiency: 90
      soc_min: 10
      soc_max: 90
    multi_parameters:
      max_energy: [1000, 1500] # List
      charge_efficiency: range(80, 90, 10) # range (min=80, max=90, step=5)
    inputs:
      ...
    outputs:
      ...
    states:
      ...
```

#### Lookup Table and Output Files
When `run_parallel_file()` executes:
- A file named `scenariotable.csv` is automatically generated in the output directory. This file maps each simulation ID to its corresponding parameter values, allowing easy reference between results and scenario configuration.
- In the YAML scenario file, **only one output file should be defined** under the `monitor` section. The Illuminator will automatically append the simulation ID number to the output filename, ensuring results from different simulations do not overwrite each other.

For example:
```yaml
monitor:
  output_file: "results/simulation_output.csv"
```
will produce:
```
results/simulation_output_1.csv
results/simulation_output_2.csv
results/simulation_output_3.csv
...
```

## Run locally

You can run Illuminator simulations in parallel in your machine if **OpenMPI** (or another MPI implementation) is installed and accessible to Python via `mpi4py`.

<details>
<summary> Installation of OpenMPI</summary>


**Linux**
```shell
sudo apt install openmpi-bin libopenmpi-dev
```

**MacOS**
```shell
brew install open-mpi
```

**Windows**
1. Install Microsoft MPI (MS-MPI) from the [Microsoft's website](https://learn.microsoft.com/en-us/message-passing-interface/microsoft-mpi)
2. Add the MPI `bin` directory (e.g., `C:\Program Files\Microsoft MPI\Bin`) to your system `PATH`

</details>

```shell
# Clone the repository
git clone https://github.com/Illuminator-team/Illuminator.git
cd Illuminator

# Checkout the dev-mpi-prallel branch (feature currently only available there)
git checkout dev-mpi-parallel

# Install the dependencies in a Conda environment
conda env create -f environment.yml
conda activate illuminator

# Uninstall the illuminator that comes from environment.yml (from PyPi)
pip uninstall illuminator

# Build and install illuminator from the dev-mpi-parallel branch
pip install .

# Run all simulations defined in a multi-parameter scenario file
mpirun -np 4 illuminator scenario run_parallel myscenario.yaml
```

## Run in DelftBlue

Although the following example is tailored for the DelftBlue cluster, the setup can be easily adapted for any other HPC clusters. The example demonstrates how to submit a SLURM job that executes the Illuminator's run_parallel command.

### Initial setup

The initial setup of configuring conda and creating the illuminator environment need to be done only once. 

```shell
# Connect to the cluster
ssh netid@login.delftblue.tudelft.nl

# Load miniconda module
module load miniconda3

# Optional: make conda install packages in /scratch instead of /home
# https://doc.dhpc.tudelft.nl/delftblue/howtos/conda/
mkdir -p /scratch/${USER}/.conda
ln -s /scratch/${USER}/.conda $HOME/.conda

# Clone the Illuminator repository
cd scratch/${USER} # optional: work in the /scratch partition as /home only has 30GB of storage per user
git clone https://github.com/Illuminator-team/Illuminator.git
cd Illuminator

# Checkout the dev-mpi-prallel branch (parallel feature currently only available there)
git checkout dev-mpi-parallel

# Install the dependencies in a Conda environment
conda env create -f environment.yml
conda activate illuminator

# Uninstall the illuminator that comes from environment.yml (from PyPi)
pip uninstall illuminator

# Build and install illuminator from the dev-mpi-parallel branch
pip install .


```

### Submit a job

Create a multi-parameter scenario file, e.g. `myscenario.yml`. In the same directory, create a job submission script, e.g. `run_illuminator.sh`:
```
#!/bin/bash
#SBATCH --job-name="illum"
#SBATCH --time=00:30:00        # max job runtime in hh:mm:ss
#SBATCH --ntasks=1             # num of processes (MPI ranks)
#SBATCH --cpus-per-task=1      # num threads per MPI process (irrelevant for now)
#SBATCH --mem-per-cpu=1G
#SBATCH --partition=compute
 
# Load required modules:
module load miniconda3
module load 2025
module load openmpi
 
# Reset any pre-existing conda nesting
# https://doc.dhpc.tudelft.nl/delftblue/Python/#conda
unset CONDA_SHLVL
source "$(conda info --base)/etc/profile.d/conda.sh"
 
# Activate illuminator conda environment
#conda env create -f ../../environment.yml
conda activate illuminator
 
# Run simulations in multi-parameter scenario file, in parallel using MPI
srun illuminator scenario run_parallel myscenario.yml
 
# Deactivate environment
conda deactivate
 
# Optional: print job efficiency summary
seff $SLURM_JOB_ID
```

Submit the job with:
```
sbatch run_illuminator.sh
```

## Tests

Tests to ensure the feature behaves as expected were added to `tests/parallel_scenarios`. A local **OpenMPI** (or another MPI implementation) installation is needed to run the tests. All the data needed to run the tests is located in tests/parallel_scenarios/data.

```shell
# Clone the repository
git clone https://github.com/Illuminator-team/Illuminator.git
cd Illuminator

# Checkout the dev-mpi-prallel branch (feature currently only available there)
git checkout dev-mpi-parallel

# Install the dependencies in a Conda environment
conda env create -f environment.yml
conda activate illuminator

# Uninstall the illuminator that comes from environment.yml (from PyPi)
pip uninstall illuminator

# Build and install illuminator from the dev-mpi-parallel branch
pip install .

# Run the tests
pytest tests/parallel_scenarios
```

