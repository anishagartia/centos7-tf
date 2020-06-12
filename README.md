# centos7-tf
Guide to build tensorflow from source on CentOS 7

Recipe to build Intel Tensorflow 2.1 with Bfloat16 support and run Resnet50 v1.5.
About BFloat16: https://software.intel.com/content/www/us/en/develop/download/bfloat16-hardware-numerics-definition.html

(Following instructions have been tested on CentOS 7.8)

Contents
## Install Tensorflow 2.1 with BF16 optimizations	1
## Run Resnet50 Bfloat16 batch inference Instructions	2
## Run Resnet50 Bfloat16 training Instruction	2


I. Install Tensorflow 2.1 with BF16 optimizations

conda Install  -y numpy keras-applications keras-preprocessing pip six wheel mock
git clone https://github.com/Intel-tensorflow/tensorflow.git
git checkout bf16/base
 
## Option 1 Install latest Bazel :
##https://docs.bazel.build/versions/master/install.html

wget https://copr.fedorainfracloud.org/coprs/vbatts/bazel/repo/epel-7/vbatts-bazel-epel-7.repo

cp vbatts-bazel-epel-7.repo /etc/yum.repos.d

install -y bazel3

vi .bazelversion
##Change to 3.2.0
	Option2 Install Bazel 3.0.0:
wget https://copr-be.cloud.fedoraproject.org/results/vbatts/bazel/srpm-builds/01330079/bazel3-3.0.0-1.fc31.src.rpm
rpmbuild --rebuild  bazel3-3.0.0-1.fc31.src.rpm

cd ~/rpmbuild/RPMS/x86_64

sudo yum install bazel3.x86_64 0:3.0.0-1.el7.rpm

rpm -ivh bazel3-3.0.0-1.el7.x86_64.rpm


cd tensorflow
./configure
 
bazel build --config=mkl --config=opt --copt=-mfma --copt=-DENABLE_INTEL_MKL_BFLOAT16 -j 16 //tensorflow/tools/pip_package:build_pip_package
 
bazel-bin/tensorflow/tools/pip_package/build_pip_package ~/path_to_save_wheel
pip install --upgrade --user ~/path_to_save_wheel/<wheel_name.whl>

##Tensorflow successfully Installed!!
 
II. Run Resnet50 Bfloat16 batch inference Instructions

wget https://storage.googleapis.com/intel-optimized-tensorflow/models/v1_6_1/resnet50_v1_5_bfloat16.pb -P <target path>
git clone https://github.com/IntelAI/models.git
git checkout v1.6.1
$ cd <git clone folder path>/models/benchmarks
$ python launch_benchmark.py \
    --in-graph <target path>/resnet50_v1_bfloat16.pb \
    --model-name resnet50v1_5 \
    --framework tensorflow \
    --precision bfloat16 \
    --mode inference \
    --batch-size=128 \
    --socket-id 0 \

III. Run Resnet50 Bfloat16 training Instruction
https://github.com/IntelAI/models/tree/v1.6.1/benchmarks/image_recognition/tensorflow/resnet50v1_5#bfloat16-training-instructions

