# TOO - Tensorflow On Odroid

Welcome to TOO, Tensorflow On Odroid.

In this repository you will find builds for tensorflow on odroid running Ubuntu.
The main goal of this project is to provide simple wheel packages of tensorflow to install on odroid
and setup a data science environment.

All the commands/patch are inspired and extracted from the following pages:
 - https://docs.bazel.build/versions/master/install-compile-source.html#bootstrap-bazel
 - https://www.tensorflow.org/install/source
 - https://github.com/robsanpam/TensorFlow_From_Sources
 - https://github.com/grpc/grpc/issues/18689
 - http://lists.busybox.net/pipermail/buildroot/2019-September/259487.html
 - https://github.com/andrewor14/tensorflow/commit/8e52b52c63c599a48d42f55c1c29506d6d8d2f0e
 - https://github.com/tensorflow/tensorflow/issues/15866
 - https://gist.github.com/EKami/9869ae6347f68c592c5b5cd181a3b205
 - https://medium.com/analytics-vidhya/iot-bazel-1-1-0-on-odroid-xu4-f38e4825c78e
 - https://stackoverflow.com/questions/46653465/tensorflow-compilation-on-odroid-xu4
 - https://devtalk.nvidia.com/default/topic/1032872/tensorflow-1-7-whl-issue-on-tx2/
 - https://stackoverflow.com/questions/55190272/java-lang-outofmemoryerror-when-running-bazel-build
 - https://github.com/bazelbuild/bazel/issues/1308

What I did?
  - pickup the best ideas
  - reads the codes
  - reads and understand the compilation errors
  - mix everythings in one single solution

## Releases 

ubuntu 18.04 armv7l
-------------------

[tensorflow-1.14.1-cp36-cp36m-linux_armv7l.whl](https://github.com/gmrandazzo/TOO/blob/master/packages/wheel/tensorflow-1.14.1-cp36-cp36m-linux_armv7l.whl)
[tensorflow-2.0.1-cp36-cp36m-linux_armv7l.whl](https://github.com/gmrandazzo/TOO/blob/master/packages/wheel/tensorflow-2.0.1-cp36-cp36m-linux_armv7l.whl)


## Install tensorflow

* install the dependencies
```
git clone https://github.com/gmrandazzo/TOO.git
cd TOO/packages/wheel
pip3 install --user scipy-1.4.0-cp36-cp36-linux_armv7l.whl 
apt install libopenmpi2 openmpi-bin openmpi-common
```


* if you whant tensorflow 1.14 please use this command:

```
pip3 install --user tensorflow-1.14.1-cp36-cp36m-linux_armv7l.whl
```

if you whant tensorflow 2.0 please use this command:
```
pip3 install --user tensorflow-2.0.1-cp36-cp36m-linux_armv7l.whl
```


## Install keras only for tensorflow 1.14!
To install keras you need a recent scipy package. For that reason you can also find in this repository 
one of the recent scipy wheel package. Please download and install it as user

```
pip3 install --user keras
```

# Compile tensorflow 1.14/2.0 on Ubuntu 18.04

To compile tensorflow you need at list 8GB of memory.
This means that your odroid is proided by 4GB of ram.
Then you need a swap of 4 GB. My advice is to have the double of the ram so 8GB of swap.
Hence you have to create an additional swap file of 8 GB using fallocate or dd. 
Then mount with swapon as root.

```
pip3 install six numpy wheel setuptools mock 'future>=0.17.1'
pip3 install keras_applications --no-deps
pip3 install keras_preprocessing --no-deps
```

* install gcc-8
```
apt install gcc-8 libgcc-8-dev

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 10 \
    --slave /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-7 \
    --slave /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-7 \
    --slave /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-7

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 20 \
    --slave /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-8 \
    --slave /usr/bin/gcc-nm gcc-nm /usr/bin/gcc-nm-8 \
    --slave /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-8

sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 20
```

* select gcc-8
```
sudo update-alternative --config gcc
```

* install openjdk-11
```
apt install openjdk-11-jre openjdk-11-jdk
```

* select openjdk-11
```
sudo update-alternatives --config java 
```

* compile bazel 0.26.1 with the bootstrap procedure
```
wget https://github.com/bazelbuild/bazel/releases/download/0.26.1/bazel-0.26.1-dist.zip

mkdir bazel-0.26.1
unzip -e bazel-0.26.1-dist.zip -d bazel-0.26.1
tar -xf bazel-0.26.1.tar.gz
cd bazel-0.26.1
ulimit -c unlimited
env BAZEL_JAVAC_OPTS="-J-Xms1024m -J-Xmx2048m" EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh

cd output
sudo cp bazel /usr/local/bin
```

* or copy the bazel 0.26 binary version that you can find here in the binary directory
```
cp packages/binary/bazel-0.26.1-armv7l /usr/local/bin
```


* install tensorflow/keras dependencies
```
apt install libopenmpi-dev libopenmpi2 openmpi-bin openmpi-common
```

* download tensorflow 
```
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
```
* then chekcout r1.14 and apply the patch to avoid compilation failures

```
git checkout r1.14 
patch -p1 < <path where is the patch>/tf_r1.14_armv7l.patch 
```

* or checkout the new 2.0 version and apply the apposite patch to avoid compilation failure
```
git checkout r2.0
patch -p1 < <path where is the patch>/tf_r2.0.1_armv7l.patch
```


* Run configure and setup the followig variables:
```
./configure


WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
You have bazel 0.26.1- (@non-git) installed.
Please specify the location of python. [Default is /usr/bin/python]: /usr/bin/python3


Found possible Python library paths:
  /usr/lib/python3/dist-packages
  /usr/local/lib/python3.6/dist-packages
Please input the desired Python library path to use.  Default is [/usr/lib/python3/dist-packages]

Do you wish to build TensorFlow with XLA JIT support? [Y/n]: Y
XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: N
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: N
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with ROCm support? [y/N]: N
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: N
No CUDA support will be enabled for TensorFlow.

Do you wish to download a fresh release of clang? (Experimental) [y/N]: n
Clang will not be downloaded.

Do you wish to build TensorFlow with MPI support? [y/N]: Y
MPI support will be enabled for TensorFlow.

Please specify the MPI toolkit folder. [Default is /usr]: /usr/lib/arm-linux-gnueabihf/openmpi/


Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: -march=native -Wno-sign-compare


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: N
Not configuring the WORKSPACE for Android builds.

```

* Modify the .tf_configure.bazelrc configure file by adding TF_NEED_S3="0"

```
build --action_env PYTHON_BIN_PATH="/usr/bin/python3"
build --action_env PYTHON_LIB_PATH="/usr/lib/python3/dist-packages"
build --python_path="/usr/bin/python3"
build:xla --define with_xla_support=true
build --action_env TF_NEED_OPENCL_SYCL="0"
build --action_env TF_NEED_ROCM="0"
build --action_env TF_NEED_CUDA="0"
build --action_env TF_DOWNLOAD_CLANG="0"
build --action_env TF_NEED_S3="0"
build:None --define with_mpi_support=true
build --config=None
build:opt --copt=-march=native
build:opt --copt=-Wno-sign-compare
build:opt --host_copt=-march=native
build:opt --define with_default_optimizations=true
build:v2 --define=tf_api_version=2
test --flaky_test_attempts=3
test --test_size_filters=small,medium
test --test_tag_filters=-benchmark-test,-no_oss,-oss_serial
test --build_tag_filters=-benchmark-test,-no_oss
test --test_tag_filters=-gpu
test --build_tag_filters=-gpu
build --action_env TF_CONFIGURE_IOS="0"
```

* Build with this command
```
bazel --host_jvm_args="-Xms1024m" \ 
      --host_jvm_args="-Xmx2048m" \
      build -c opt \
      --copt="-funsafe-math-optimizations" \
      --copt="-ftree-vectorize" \
      --copt="-fomit-frame-pointer" \
      --config=noaws \
      --config=noignite \
      --config=nokafka \
      --verbose_failures \
      tensorflow/tools/pip_package:build_pip_package
```


* create the wheel package
```
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```
Then in /tmp/tensorflow_pkg you will find the final package to install with pip3.


* compile scipy 1.4.0
```
apt install libblas-dev liblapack-dev python3-pybind11 pybind11-dev
apt install build-essential libatlas-base-dev gfortran libfreetype6-dev python3-setuptools
apt install protobuf-compiler libprotobuf-dev openssl libssl-dev libcurl4-openssl-dev

pip3 install --user cython
pip3 install --user numpy

dpkg -i pybind11-dev_2.3.0-2_all.deb python3-pybind11_2.3.0-2_all.deb

git clone https://github.com/scipy/scipy.git
cd scipy
git checkout v1.4.0
python3 setup.py bdist_wheel
pip3 install --user dist/scipy-1.4.0-cp36-cp36m-linux_armv7l.whl

```

## Author and acknowledgments
 - Giuseppe Marco Randazzo <gmrandazzo AT gmail DOT com>

