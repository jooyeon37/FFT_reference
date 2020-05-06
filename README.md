# CUDA FFT Reference code
This is the reference [Fast Fourier Transform(FFT)][fft_release] code for computer architecture class 2020.\
Find "ExecFft" function in src/ffthelper.cu, where the radix-2 [Cooley-Tukey FFT][cooley-tukey] algorithm is implemented as a reference code. You are asked to change the function to optimize for the GPU architecture, using the [GPGPU-sim simulator][gpgpu-sim].


[class_page]: https://scale-snu.github.io/jekyll/update/2020/03/16/aca2020-lecture-01.html
[docker_image]: https://hub.docker.com/r/michael604/comparch_430.636_spring2020
[gpgpu-sim]: https://github.com/gpgpu-sim/gpgpu-sim_distribution
[fft_release]: https://github.com/gajh-classes/FFT_reference
[cooley-tukey]: https://en.wikipedia.org/wiki/Cooley%E2%80%93Tukey_FFT_algorithm
[machine_config]: https://github.com/gpgpu-sim/gpgpu-sim_distribution/tree/dev/configs/tested-cfgs/SM6_TITANX


## Overview
In this project, you are recommended to use the docker to set up the GPGPU-sim and execution environment. You will optimize your SW code or HW using the [GPGPU-sim simulator][gpgpu-sim]. 

The set-up for the project will be as follows. 
1. Pull docker images
2. Build FFT_release
3. set environmental variables(for [GPGPU_sim][gpgpu-sim])
4. Execute the FFT_release application binary


## Install Docker & Pull Image
The following instructions are based on the Ubuntu 18.04 LTS. You may need *sudo* authority to build and run the docker container.

```
$ apt-get install docker.io
$ docker pull michael604/scal2020_gpgpu_sim
$ docker pull michael604/comparch_430.636_spring2020
$ docker run -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined michael604/comparch_430.636_spring2020 --name {NAME}
```
Necessary dependencies and properly built [GPGPU-sim][gpgpu-sim] (dev 4.0) is already included in the [docker image][docker_image].
You need to create the container with `-it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined` option if you want to use gdb inside the container. You may name your container with `--name {NAME}` option.
GPGPU-sim is built as a release option in the provided image, but you may recompile it according to your need.


## How to Build & Run
This part will explain the necessary procedures to properly build [FFT_release][fft_release] CUDA application and how to attach it to GPGPU-sim when executing the binary.\
We also provide automated script file `makestep.sh`, so you may also use the script.\
(Directory names in the following explanation is based on the [docker image][docker_image] that is provided.

### 1. Build
in `/root/FFT_release` directory,
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
export LD_LIBRARY_PATH=/usr/local/cuda/lib64
make -j
```
Binary file named `fft_release` will be generated in `/root/FFT_release/build`

### 2. Run
You need to set the correct environment variable to properly attach the CUDA application to GPGPU-sim. You also have to locate the correct GPGPU-sim configuration files and matrix data.

##### Set Environmental Variable
You first need to set environmental variables correctly.\
In `/root/gpgpu-sim_distribution` directory,
```bash
$ source setup_environment {BUILD_TYPE}
```
The default `BUILD_TYPE` in the [docker image][docker_image] is release.

##### Locate Configuration Files & Matrix Data
You need to locate `config_{ARCH}_islip.cnt` & `gpgpusim.config` files from `/gpgpu-sim_distribute/configs/tested-cfgs`, to the directory that you are going to run the application.

In `/root/FFT_release` directory,
```bash
$ mkdir build/run && cd build/run
$ cp /root/gpgpu-sim_distrubute/configs/tested-cfgs/{SM_NUM}/* ./
$ cp /root/FFT_release/test/*.txt ./
```
We will use [`SM6_TITANX`][machine_config] configuration as a default machine configuration for the [GPGPU-sim][gpgpu-sim]. We do not need to run power simulation, so you need to change the `power_simulation_enabled` value to 0 in `gpgpusim.config` file.

##### Run CUDA Application
Normally execute the application binary, and it will attach to GPGPU-sim properly.
```bash
cd /root/FFT_release/build/run
./../fft_release
```

##### makestep.sh
in `/root/FFT_release`, automated script is provided. You may use the script to skip the above-explained procedures.



## Implementation & Evaluation
You can take either of the following two directions in conducting
this term project:

1. SW only approach: Optimize the **FFT / iFFT** code with higher performance in the given `SM6_TITANX` Pascal configuration. You can modify every part of the code and add any additional custom functions. However, it is necessary to explain your implementation in your report. 

2. HW modification: Modify the machine description file(`SM6_TITANX`) or GPGPU-sim simulator to suggest and implement a better hardware.

* When you run the application binary with GPGPU-sim, `gpgpu_simulation_total_cycle` is displayed at the end of the log. The performance of your optimized code and application will be evaluated accordingly. 

* If it's necessary, you are allowed to modify the CMakeLists.txt. However, do not modify any important flags, such as optimization level flags. You also have to notify and explain what you have changed.



## Report

1. SW only approach: Describe your method of code optimization.

2. HW modification: Describe your suggested hardware, and the reasoning behind the modifications. 

* If you have not used the provided [docker image][docker_image], please specify the environment that you have worked in.

## Contact

If you find any errors or issues in the project, feel free to e-mail us. Our e-mails are as below, and are also posted on our [class description page][class_page].

Michael Jaemin Kim: michael604@scale.snu.ac.kr\
Sangpyo Kim: spkim@scale.snu.ac.kr\
