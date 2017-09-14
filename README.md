#Google cloud compute engine

1. visit https://cloud.google.com
	- click on try it free
	- create an account/use an existing google account
2. Create a new project if you do not have one
3. Select compute engine on the menu panel on the left
	- create instance
	- use a GPU enabled zone if you want to use GPU for tensorflow model building 
		- at the time of writing this post us-east1-c was GPU enabled
	- select a CPU type
		- select a micro (1 shared vCPU) for now
			- setting up the tensorflow does not require high end hardware, thus we can save on some cost
			- at the time of writing this post micro CPU usage was free

	- select bootdisk 
		- use ubuntu 16.04 LTS distribution
		- 20GB persistent disk should be enough (you can increase it if you want to)
	- allow http and https traffic 

##Setting up environment
1. Install java
	install java
	sudo apt-get install default-jre
	sudo apt-get install default-jdk
	java -version

2. Install gcc
	
	These commands are based on a askubuntu answer http://askubuntu.com/a/581497
	To install gcc-6 (gcc-6.1.1), I had to do more stuff as shown below.
	USE THOSE COMMANDS AT YOUR OWN RISK. I SHALL NOT BE RESPONSIBLE FOR ANYTHING.
	ABSOLUTELY NO WARRANTY.

	sudo apt-get update && \
	sudo apt-get install build-essential software-properties-common -y && \
	sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y && \
	sudo apt-get update && \
	sudo apt-get install gcc-snapshot -y && \
	sudo apt-get update && \
	sudo apt-get install gcc-6 g++-6 -y && \
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 60 --slave /usr/bin/g++ g++ /usr/bin/g++-6 && \
	sudo apt-get install gcc-4.8 g++-4.8 -y && \
	sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8;

	When completed, you must change to the gcc you want to work with by default. Type in your terminal:
	sudo update-alternatives --config gcc

	To verify if it worked. Just type in your terminal
	gcc -v

3. Install bazel

	echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
	curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
	sudo apt-get update && sudo apt-get install bazel
	sudo apt-get upgrade bazel

4. Now before we continue with the installation, lets upgrade our instance specifications :
	- shutdown the instance
	- go to : https://console.cloud.google.com/compute/instances
	- click on the name of the instance you are using
	- click on 'edit' option
	- here customize the CPU, here is how I upgraded the system (you could do the same or different depending on your requirement)
		- upgrade to CPU to n1-standard-4 (4 vCPUs, 15 GB memory)
		- add GPU
			- GPU can only be added if you have upgraded your account (click here to learn more about adding GPU quota to your project)
	- save and start the instance again

5. Install CUDA : 

	- depending on your architecture download CUDA for linux from : https://developer.nvidia.com/cuda-downloads

	- to upload run file to your instance use google cloud bucket (click here to know more)
		- use : $ gsutil cp -r gs://my-bucket/ .
			- to copy file from bucket storage to instance

	- check for a GPU : $ lspci | grep -i nvidia

	- check if nouveau drivers are loaded : $ lsmod | grep nouveau

	- Disable if loaded : 

		Create a file at /etc/modprobe.d/blacklist-nouveau.conf with the following contents:
		blacklist nouveau
		options nouveau modeset=0
		Regenerate the kernel initramfs:
		$ sudo update-initramfs -u

	- $ sudo sh cuda_<version>_linux.run
		- use space to go through the EULA faster
		- install driver , CUDA and CUDA samples

	- post installation add the following to profile 

		$ nano ~/.profile

		PATH="/usr/local/cuda-X.Y/bin:$PATH"
		LD_LIBRARY_PATH="/usr/local/cuda-X.Y/lib64:$LD_LIBRARY_PATH"

6. installing CUDNNA
	- download cudnna tar from https://developer.nvidia.com/rdp/cudnn-download
		- default version supported at the time of writing this post was 6.0
		- however I installed 5.1.10

	- to upload run file to your instance use google cloud bucket (click here to know more)
			- use : $ gsutil cp -r gs://my-bucket/ .
				- to copy file from bucket storage to instance

	- 	Execute the below commands
		$ sudo tar -xzvf cudnn-8.0-linux-x64-vX.Y.tgz
		$ sudo cp -P cuda/include/cudnn.h /usr/local/cuda/include #maintains symbolic link
		$ sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64 #maintains symbolic link
		$ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

	- Add this line at the end of ~/.bashrc
		export CUDA_HOME=/usr/local/cuda

	- $ source ~/.bashrc

	- Add to /etc/ld.so.conf.d/cuda-8-0.conf :

		/usr/local/cuda/lib64
		/usr/local/cuda/extras/CUPTI/lib64 

		- Run sudo ldconfig

7. Installing tensorflow

	- I would recommend installing tensorflow by building it from sources : 
			git clone https://github.com/tensorflow/tensorflow 
			cd tensorflow
			git checkout


	- sudo apt-get install python-numpy python-dev python-pip python-wheel
										OR 
		sudo apt-get install python3-numpy python3-dev python3-pip python3-wheel

	- GPU installation pre-requistie
		$ sudo apt-get install libcupti-dev 

	- start configuring the tensorflow installation
		cd tensorflow
		./configure

	- following is a snapshot of configuration from : tensorflow.org
	'''

	Please specify the location of python. [Default is /usr/bin/python]: /usr/bin/python2.7
	Found possible Python library paths:
	  /usr/local/lib/python2.7/dist-packages
	  /usr/lib/python2.7/dist-packages
	Please input the desired Python library path to use.  Default is [/usr/lib/python2.7/dist-packages]

	Using python library path: /usr/local/lib/python2.7/dist-packages
	Do you wish to build TensorFlow with MKL support? [y/N]
	No MKL support will be enabled for TensorFlow
	Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
	Do you wish to use jemalloc as the malloc implementation? [Y/n]
	jemalloc enabled
	Do you wish to build TensorFlow with Google Cloud Platform support? [y/N]
	No Google Cloud Platform support will be enabled for TensorFlow
	Do you wish to build TensorFlow with Hadoop File System support? [y/N]
	No Hadoop File System support will be enabled for TensorFlow
	Do you wish to build TensorFlow with the XLA just-in-time compiler (experimental)? [y/N]
	No XLA support will be enabled for TensorFlow
	Do you wish to build TensorFlow with VERBS support? [y/N]
	No VERBS support will be enabled for TensorFlow
	Do you wish to build TensorFlow with OpenCL support? [y/N]
	No OpenCL support will be enabled for TensorFlow
	Do you wish to build TensorFlow with CUDA support? [y/N] Y
	CUDA support will be enabled for TensorFlow
	Do you want to use clang as CUDA compiler? [y/N]
	nvcc will be used as CUDA compiler
	Please specify the Cuda SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 8.0]: 8.0
	Please specify the location where CUDA 8.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
	Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]:
	Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 6.0]: 6
	Please specify the location where cuDNN 6 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:
	Please specify a list of comma-separated Cuda compute capabilities you want to build with.
	You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
	Please note that each additional compute capability significantly increases your build time and binary size.
	[Default is: "3.5,5.2"]: 3.0
	Do you wish to build TensorFlow with MPI support? [y/N] 
	MPI support will not be enabled for TensorFlow
	Configuration finished
	'''
	- Now that configuration is done, use bazel to build
		- $ bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package
		- $ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
		- $ sudo pip install /tmp/tensorflow_pkg/tensorflow-1.3.0-py2-none-any.whl


7. Validate installation : 
	- Activate python shell : 
		$ python
	- code in the following : 
		import tensorflow as tf
		hello = tf.constant('Systems up and running')
		sess = tf.Session()
		print(sess.run(hello))
	- the output should be : Systems up and running
