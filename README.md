# TX2-Tensorflow-Jetpack-3.1-
RX2 install Tensorflow
  
### reference:  
https://home.gamer.com.tw/creationDetail.php?sn=3988409  
https://note.youdao.com/ynoteshare1/index.html?id=3779eaea012bfec0fd1dc3e6b27a2b62&type=note#/  
https://zhuanlan.zhihu.com/p/48406518  
  
### Enviroment  
  
#version  
CUDA 9.0  
CuDnn 7.0.5  
Tensorflow 1.5  
  
#查CUDA版本(Nvidia CUDA Compiler)  
> nvcc --version  
> cat /usr/local/cuda/version.txt  
  
#查cuDNN版本  
> cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2  
> cat /usr/include/cudnn.h | grep CUDNN_MAJOR -A 2  
   
#查看CPU、GPU溫度及使用率等資訊  
sudo ~/tegrastats  
(用來方便看執行的東西有沒有在用GPU跑等等)  
  
###建置swapfile  
$ git clone https://github.com/jetsonhacks/postFlashTX1  
$ cd postFlash/  
$ sudo ./creatSwapfile.sh –d ‘ /media/ubuntu/JetsonSSD’ –s 10 –a  
(-a   auto Enable swap on boot in /etc/fstab)  
  
在選單上 進disk -> Edit Mount Options…  
-> Automatic改成off -> mount point: /media/nvidia/JetsonSSD  
-> Identify as 選擇 UUID  
(重開機)  
  
###安裝JDK (目前版本是1.8.0_171，可用 $ java -version 查看)  
$ sudo add-apt-repository ppa:webupd8team/java  
$ sudo apt-get update  
$ sudo apt-get install oracle-java8-installer  
  
###安裝Python依賴  
$ sudo apt-get install zip unzip autoconf automake libtool curl zlib1g-dev maven –y  
$ sudo apt-get install python-numpy swig python-dev python-pip python-wheel -y  
(python在刷機過程已安裝好   安裝linux內附python)  

###安裝 bazel 版本 0.11.1
$ wget --no-check-certificate https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel-0.11.1-dist.zip  
$ unzip bazel-0.11.1-dist.zip -d bazel-0.11.1-dist  
$ cd ./bazel.0.11.1-dist  
$ ./compile.sh  
(編譯過程有錯誤 有可能為bazel 版本比對bug後面會提到)  
$ sudo cp output/bazel /usr/local/bin  
  
###下載 TensowFlow 版本1.5  
$ cd  
$ git clone https://github.com/tensorflow/tensorflow.git  
$ cd ./tensorflow  
$ git checkout v1.5.0  
(切換分支版本)  
  
$ sudo gedit tensorflow/stream_executor/cuda/cuda_gpu_executor.cc  
#在function開始的地方 添加 (我的是在第857行)  
     LOG(INFO) << "ARM has no NUMA node, hardcoding to return zero";  
     return 0;  
     
###編譯 TensowFlow 版本1.5  
$ sudo mkdir /usr/lib/aarch64-linux-gnu/include/  
$ sudo cp /usr/include/cudnn.h /usr/lib/aarch64-linux-gnu/include/cudnn.h  
  
$ ./configure  
#配置路徑、support等  (參見http://jksoftcn.com/xue-xi-tensorflow1.html)  
TX2:配置  
Please specify the location of python. [Default is /usr/bin/python]: /usr/bin/python3  
Do you wish to build TensorFlow with CUDA support? [y/N] y  
Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to use system default]: 7.0.5  
Please specify the location where CUDA  toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:   
Please specify the location where cuDNN  library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: /usr  
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 6.1]: 6.2  
Do you wish to build TensorFlow with OpenCL support? [y/N]: N   #這個必須選N，否則會出錯  
Do you want to use clang as CUDA compiler? [y/N]: N   #這個必須選N，否則會出錯   
Do you wish to build TensorFlow with MPI support? [y/N]: N   #這個必須選N，否則會出錯  
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]: #根據官網教程，直接回車   
    
$ bazel build -c opt --local_resources 3072,4.0,1.0 --verbose_failures --config=cuda //tensorflow/tools/pip_package:build_pip_package  
(編譯過程約3個小時) 若之後要更改coufigure ，須再設定./configure後重新bazel build一次  
  
#修改bazel version verify  
#/home/nvidia/tensorflow/tensorflow/worksapce.bzl 第48行  
#/home/nvidia/.cache/bazel/_bazel_nvidia/d2751a49dacf4cb14a513ec663770624/external/io_bazel_rules_closure/closure/repositories.bzl 第69行  
修改為check_version("0.0.0")  
  
###TensorFlow Package (Generate a wheel)  
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg  
$ mv /tmp/tensorflow_pkg/tensorflow-1.5.0-cp27-cp27mu-linux_aarch64.whl $HOME  
$ sudo pip install $HOME/tensorflow-1.5.0-cp27-cp27mu-linux_aarch64.whl  
(for python 2.x)  
  
###TensorFlow & 確認TF version  
#cd到其他目錄，在tensorflow目錄下import tensorflow  
#會出現 ImportError: No module named platform  
cd  
python  
>>> import tensorflow as tf  
>>> tf.__version__  
>>> 1.5.0  
   
有成功出現代表安裝成功了~  

###安裝Keras  
$ sudo apt-get install python-scipy  python-numpy python-pil python-matplotlib libpng-dev python-decorator python-imaging libblas-common libblas3 libgfortran3 liblapack3 libgdf5-dev libhdf5-dev  
$ pip install --upgrade pip  

#目前 pip-10.0.1   pip有bug  參見上面  
$ sudo pip install h5py  
$ sudo pip install keras  
