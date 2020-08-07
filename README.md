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

Note: Only Linux (Ubuntu 16.04, 18.04) install has been tested. Instructions for other operating systems are from the pytorch repo and are left here for your convenience.

Install
- [NVIDIA CUDA](https://developer.nvidia.com/cuda-downloads) 7.5 or above
- [NVIDIA cuDNN](https://developer.nvidia.com/cudnn) v6.x or above

If you want to build on Windows, Visual Studio 2017 and NVTX are also needed.

### GPGPU-Sim Setup
Follow steps 1 and 2 [here](https://github.com/gpgpu-sim/gpgpu-sim_distribution#installing-building-and-running-gpgpu-sim).
### Get the PyTorch source
```bash
git clone [this repo]
cd pytorch-gpgpu-sim
git -c submodule."third_party/nervanagpu".update=none submodule update --init
```
The Nervana repository no longer exists, so `git clone --recursive` can not be used. To work around this, we manually disable pulling from that particular repository, while updating the rest of the modules.


### Install PyTorch
On Linux
#### Setting Environmental Variables
The following variables must be set:
```
export CUDNN_INCLUDE_DIR=<path of directory where cudnn.h is located>
```
```
export CUDNN_LIBRARY=<path of static cudnn library>
```
These variables let Pytorch know where to look for CuDNN files during installation.
```
export TORCH_CUDA_ARCH_LIST="6.1+PTX"
```
This variable sets the sm and compute flags used to build anything CUDA related. Without setting it, a script will be ran to to probe the GPU for it. It is unclear if the GPU-probing library will use GPGPU-Sim or hardware, but it cannot detect GPGPU-Sim configuration file anyways. If it does not detect any GPUs, it defaults to a full list, which includes “6.1+PTX” (the flag we want.) Manually setting this will reduce the number of PTX files required to parse later.
#### Installing Dependent Libraries
If Torchvision is not installed, install that first. This will also install NumPy if it was not installed already, as well as the latest version of Pytorch. Since the current version of Pytorch that supports gpgpusim is 0.4 (as opposed to 1.x), some functionalities in the newest version of Torchvision are not compatible with the older version of pytorch. As a result, we recommend installing Torchvision 0.2.2.
```
pip install torchvision==0.2.2
```
#### Uninstalling Existing Torch Libraries
Installing Torchvision would install the latest Pytorch as well, so manually uninstall this before building the one compatible with gpgpu-sim.
```
pip uninstall torch
```
#### Changing gcc/g++ Versions
If Ubuntu 18.04 is used, either disable the gcc warnings in CUDA, or modify the CMake file `cmake/MiscCheck.cmake` to disable the check for gcc versions (specifically lines 29-33 [here](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/blob/master/cmake/MiscCheck.cmake#L29-L33).)

After modifying the CMake file, you may have to update your PATH variable to point to a directory with gcc-5 and g++-5 (if the gcc/g++ in /usr/bin is a newer version of gcc/g++. This would disable warnings from `cudnn.h`. 

Next, you may have to modify `tools/build_pytorch_lib.sh` to specify the CUDA host compiler. This can be done by adding `-DCUDA_HOST_COMPILER='<path to gcc-5>' \` to the set of flags at every line that starts with `${CMAKE VERSION}` (more specifically [here](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/blob/master/tools/build_pytorch_libs.sh#L154) and [here](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/blob/master/tools/build_pytorch_libs.sh#L254). For the latter, the script warns you to add flags elsewhere, but it should work regardless.)  This would disable the warnings from cmake when it uses nvcc. 

Next, since we manually specified which gcc version to use, cmake might not pick up on it and may include compiler flags not supported in gcc-5, so line 231 in `CMakeLists.txt` may need to be commented out ([here](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/blob/master/CMakeLists.txt#L231).)

Lastly, to enable running other algorithms, cuBLAS must be statically linked (for example, FFT directly calls cuBLAS kernels), which can be done by changing line 255 to the path to the static cuBLAS library in `cmake/public/cuda.cmake` ([here](https://github.com/gpgpu-sim/pytorch-gpgpu-sim/blob/master/cmake/public/cuda.cmake#L255).)
#### Actually Installing Pytorch
```bash
python setup.py install
```
Use the following if you encounter root permission error:
```bash
python setup.py install --user
```
#### Linking to GPGPU-Sim
Set `PYTORCH_BIN` to point to the shared library generated by PyTorch.

```
Export PYTORCH_BIN=<pytorch directory>/torch/lib/libcaffe2_gpu.so
```
If the shared library is not in that directory, it is possible that CUDA paths were not set, and Pytorch was not built with CUDA support. In that case, uninstall the current pytorch version using pip uninstall, and restart from GPGPU-Sim setup.

#### Run Application
Try out your application. 

A tested application that ran was the MNIST example in pytorch’s repository [here](https://github.com/pytorch/examples/blob/master/mnist/main.py). For now, make sure to turn on the deterministic flag and turn off the benchmark flag (add `torch.backends.cudnn.deterministic = True` and `torch.backends.cudnn.benchmark = False` after `if __name__ == “main”`), and make sure to set iteration and epoch to 1, or else it would take potentially months before simulation completes.

#### Additional Notes
Torch's benchmark mode tries and times every convolution algorithm before picking the fastest one to use, so it would take a long time to simulate. In fact, some algorithms do not have PTX implementations, so they cannot be simulated, meaning the simulation itself may abort if benchmark mode is left on. To remedy this, make sure to turn off benchmark mode.

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

## Getting Started

Three pointers to get you started:
- [Tutorials: get you started with understanding and using PyTorch](http://pytorch.org/tutorials/)
- [Examples: easy to understand pytorch code across all domains](https://github.com/pytorch/examples)
- [The API Reference](http://pytorch.org/docs/)

