# Isambard-AI-Setup

The [official documentation](https://docs.isambard.ac.uk/) provides comprehensive instructions. 
This document focuses on a minimal setup for getting started with Isambard-AI. 
To access the system, SSH should first be configured following the official guide.

📌 More details in [user-documentation/guides/login](https://docs.isambard.ac.uk/user-documentation/guides/login/).

Before each login session, the following command should be executed:
```
clifton auth
```
Authentication expires every 12 hours.

## 1. File System Overview

Recommended file storage layout:

- Home directory (`$HOME`) → for code repositories and conda environments
- Scratch directory (`$SCRATCH`) → for datasets, checkpoints, and large files

The home directory is used for small, persistent files, while scratch is used for datasets and large files.

📌 More details in [user-documentation/information/system-storage](https://docs.isambard.ac.uk/user-documentation/information/system-storage/).

## 2. Create Conda Environment

Install Miniforge in the home directory:
```
cd $HOME
curl --location --remote-name "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```
When prompted with `conda init`, select `NO` to avoid automatic initialization conflicts.

Create an environment:
```
source ~/miniforge3/etc/profile.d/conda.sh
conda create -n wm python=3.12
```

Verify activation in a new terminal:
```
source ~/miniforge3/etc/profile.d/conda.sh
conda activate wm
```

Remove the installer:
```
rm Miniforge3-$(uname)-$(uname -m).sh
```

📌 More details in [user-documentation/guides/python](https://docs.isambard.ac.uk/user-documentation/guides/python/).

## 3. Install Packages

GPU-dependent packages should be installed inside an interactive compute session:
```
srun -N 1 --gpus 1 --pty bash
```

Activate the environment:
```
source ~/miniforge3/etc/profile.d/conda.sh
conda activate wm
```

Install PyTorch:
```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
```

📌 More details in [user-documentation/applications/ML-packages](https://docs.isambard.ac.uk/user-documentation/applications/ML-packages/).

Installation guides for commonly used packages:
- [xFormers](conda/xformers.md)

## 4. Use Jupyter Notebook

Register a kernel:
```
python -m ipykernel install --user --name wm --display-name "Python (wm)"
jupyter kernelspec list
```

Create `jupyter-user-environment.yml`:
```yml
name: jupyter-user-env
channels:
  - conda-forge
  - nodefaults
dependencies:
  # JupyterLab user interface
  - jupyterlab >=4.2,<5.0

  # Jupyter Notebook classic interface v7, built on modern Jupyter Lab & Jupyter Server
  - notebook >=7.2,<8.0
```

Create the conda environment:
```
conda env create --file jupyter-user-environment.yml
```

Create a job script `submit_jupyter_user_session_i-aip2.sh`:
```sh
#!/bin/bash
#SBATCH --job-name=jupyter_user_session
#SBATCH --gpus=1  # this also allocates 72 CPU cores and 115GB memory
#SBATCH --time=01:00:00

source ~/miniforge3/bin/activate jupyter-user-env

# Add pre-installed kernelspecs to the Jupyter data search path
export JUPYTER_PATH="/tools/brics/jupyter/jupyter_data${JUPYTER_PATH:+:}${JUPYTER_PATH:-}"

HSN_FQDN="$(hostname).hsn.ai-p2.isambard.ac.uk"
LISTEN_IP=$(dig "${HSN_FQDN}" A +short | tail -n 1)
LISTEN_PORT=8888

set -o xtrace
jupyter lab --no-browser --ip="${LISTEN_IP}" --port="${LISTEN_PORT}"
```

Submit the job:
```
sbatch submit_jupyter_user_session_i-aip2.sh
```

Check the Slurm output file:
```
+ jupyter lab --no-browser --ip=10.242.10.49 --port=8888
[I 2026-03-23 15:31:54.857 ServerApp] jupyter_lsp | extension was successfully linked.
[I 2026-03-23 15:31:54.860 ServerApp] jupyter_server_terminals | extension was successfully linked.
[I 2026-03-23 15:31:54.864 ServerApp] jupyterlab | extension was successfully linked.
[I 2026-03-23 15:31:54.867 ServerApp] notebook | extension was successfully linked.
... ...    
    To access the server, open this file in a browser:
        file:/home/u6gn/xuanya.u6gn/.local/share/jupyter/runtime/jpserver-57990-open.html
    Or copy and paste one of these URLs:
        http://10.242.10.49:8888/lab?token=7adbc69e3dca235ae966a6f7f2f562d126e376092fcafdd6
        http://127.0.0.1:8888/lab?token=7adbc69e3dca235ae966a6f7f2f562d126e376092fcafdd6
... ...
```
Interpretation of the output:
- Server IP: `10.242.10.49`
- Port: `8888`
- Token: `7adbc69e3dca235ae966a6f7f2f562d126e376092fcafdd6`

Connect from the local machine:
```
ssh -T -L localhost:<local-port>:<server-ip>:<port> <project-id>.aip2.isambard
```
Then open `http://localhost:<local-port>` and enter the token to access Jupyter Lab.

📌 More details in [user-documentation/guides/jupyter](https://docs.isambard.ac.uk/user-documentation/guides/jupyter/).

## 5. Run Tasks

Interactive tools such as `tmux` and `screen` are not supported. All jobs should be submitted via Slurm.

📌 More details in [user-documentation/guides/slurm](https://docs.isambard.ac.uk/user-documentation/guides/slurm/).
