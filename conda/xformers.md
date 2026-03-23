## Install xFormers

xFormers needs to be built against the system CUDA toolkit. 
On compute nodes, the CUDA compiler (`nvcc`) is not exposed by default. 
If CUDA is not explicitly loaded, xFormers will fall back to a CPU-only build. 
Therefore, a compatible CUDA module must be loaded before installation.

List available CUDA versions:

```sh
module avail cuda
```

Load a compatible CUDA version (e.g. CUDA 12.6) to ensure that the `nvcc` is available:
```
module load cuda/12.6
which nvcc && nvcc --version
```
The CUDA version should be compatible with the PyTorch CUDA version (e.g. cu128 → CUDA 12.x).

Reinstall xFormers from source:
```
python -m pip uninstall -y xformers
python -m pip install -U pip setuptools wheel ninja packaging numpy
export TORCH_CUDA_ARCH_LIST="9.0"
pip install -v --no-build-isolation --no-binary xformers xformers
```
- `--no-binary xformers`: forces building from source
- `TORCH_CUDA_ARCH_LIST`: specifies the target GPU architecture (`GH200` corresponds to `9.0`)

Verify installation:
```
python -m xformers.info
```

If successful, output will include information such as:
```
pytorch.version:                                   2.10.0+cu128
... ...
build.python_version:                              3.12.13
build.torch_version:                               2.10.0+cu128
build.env.TORCH_CUDA_ARCH_LIST:                    9.0
... ...
build.nvcc_version:                                12.6.77
```

This confirms that xFormers is correctly built with CUDA support and is compatible with PyTorch.
