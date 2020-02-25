# TOO
TOO - Tensorflow On Odroid
Welcome to TOO, Tensorflow On Odroid.

In this repository you will find builds for tensorflow on odroid running Ubuntu.
The main goal of this project is to provide simple wheel packages of tensorflow to install on odroid.

This repository is inspired and extracted from the following pages:
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

What I did is:
  - pickup the best ideas
  - reads the codes
  - reads and understand the compilation errors
  - mix everythings in one single solution

## Releases 

[tensorflow-1.14.1-cp36-cp36m-linux_armv7l.whl](https://github.com/gmrandazzo/TOO/blob/master/packages/tensorflow-1.14.1-cp36-cp36m-linux_armv7l.whl)


## Compile tensorflow
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
wget https://github.com/bazelbuild/bazel/archive/0.26.1.tar.gz -O bazel-0.26.1.tar.gz
tar -xf bazel-0.26.1.tar.gz
cd bazel-0.26.1
ulimit -c unlimited
env BAZEL_JAVAC_OPTS="-J-Xms384m -J-Xmx512m" EXTRA_BAZEL_ARGS="--host_javabase=@local_jdk//:jdk" bash ./compile.sh
cd output
sudo cp bazel /usr/local/bin
```

* install tensorflow/keras dependencies
```
apt install libopenmpi-dev libopenmpi2 openmpi-bin openmpi-common
apt install libblas-dev liblapack-dev python3-pybind11 pybind11-dev
```

* download tensorflow 
```
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
git checkout r1.14
```

* Write the bazel configure file
```
echo 'build --action_env PYTHON_BIN_PATH="/usr/bin/python3"
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
build --action_env TF_CONFIGURE_IOS="0"' > .tf_configure.bazelrc
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


* create the whl package
```
./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
```

Then in /tmp/tensorflow_pkg you will find the final package to install with pip3.


