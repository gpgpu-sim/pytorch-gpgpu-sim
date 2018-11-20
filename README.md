![PyTorch Logo](docs/source/_static/img/pytorch-logo-dark.png)

--------------------------------------------------------------------------------

##PyTorch for GPGPU-Sim using CUDNN
Contributor: Jonathan Lew, University of British Columbia, Dept. of Electrical and Computer Engineering

Welcome to PyTorch for GPGPU-Sim!

This repo is based on the popular PyTorch framework, release v0.4.1, the latest release as of Aug 17, 2018. It can be found at https://github.com/pytorch/pytorch

- [Installation](#markdown-header-installation)
- [GPGPU-Sim](#markdown-header-gpgpu-sim)
- [Getting Started](#markdown-header-getting-started)


## Installation

Note: Only Linux install has been tested. Instructions for other operating systems are from the pytorch repo and are left here for your convenience.

Install
- [NVIDIA CUDA](https://developer.nvidia.com/cuda-downloads) 7.5 or above
- [NVIDIA cuDNN](https://developer.nvidia.com/cudnn) v6.x or above

If you want to build on Windows, Visual Studio 2017 and NVTX are also needed.

### Get the PyTorch source
```bash
git clone --recursive [this repo]
cd pytorch-gpgpusim-install
```

### Install PyTorch
On Linux
```bash
python setup.py install
```
Use the following if you encounter root permission error:
```bash
python setup.py install --user
```

On macOS
```bash
MACOSX_DEPLOYMENT_TARGET=10.9 CC=clang CXX=clang++ python setup.py install
```

On Windows
```cmd
set "VS150COMNTOOLS=C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\VC\Auxiliary\Build"
set CMAKE_GENERATOR=Visual Studio 15 2017 Win64
set DISTUTILS_USE_SDK=1
REM The following line is needed for Python 2.7, but the support for it is very experimental.
set MSSdk=1
REM As for CUDA 8, VS2015 Update 2 or up is required to build PyTorch. Use the following two lines.
set "PREBUILD_COMMAND=%VS140COMNTOOLS%\..\..\VC\vcvarsall.bat"
set PREBUILD_COMMAND_ARGS=x64

call "%VS150COMNTOOLS%\vcvarsall.bat" x64 -vcvars_ver=14.11
python setup.py install
```
## GPGPU-Sim

1. Install GPGPU-Sim with CUDNN and PyTorch support at:

2. Follow the README to set the environment variables.

3. export PYTORCH_BIN=/path/to/libcudnn.so

## Getting Started

Three pointers to get you started:
- [Tutorials: get you started with understanding and using PyTorch](http://pytorch.org/tutorials/)
- [Examples: easy to understand pytorch code across all domains](https://github.com/pytorch/examples)
- [The API Reference](http://pytorch.org/docs/)

