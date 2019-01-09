# MNIST机器学习入门
### 实验环境
- ubuntu18 + tensorflow
- 
### 1、安装pip (Python包管理工具)

```
ktlewis@ubuntu:~/myproject$ sudo apt-get install python-pip
```

- python版本查看

```
ktlewis@ubuntu:~$ python --version
Python 2.7.15+
ktlewis@ubuntu:~$ python3 --version
Python 3.6.7
```
- 安装python --(若未安装python)

```
ktlewis@ubuntu:~$ sudo apt-get install python-pip
ktlewis@ubuntu:~$ sudo apt-get install python3-pip
```
### 2、使用pip命令安装virtualenv:
- **virtualenv**就是用来为一个应用创建一套“隔离”的Python运行环境

```
ktlewis@ubuntu:~$ sudo pip install virtualenv
Collecting virtualenv
  Downloading https://files.pythonhosted.org/packages/6a/d1/e0d142ce7b8a5c76adbfad01d853bca84c7c0240e35577498e20bc2ade7d/virtualenv-16.2.0-py2.py3-none-any.whl (1.9MB)
    100% |████████████████████████████████| 1.9MB 600kB/s 
Requirement already satisfied: setuptools>=18.0.0 in /usr/lib/python2.7/dist-packages (from virtualenv)
Installing collected packages: virtualenv
Successfully installed virtualenv-16.2.0
```
### 3、使用创建虚拟环境来运行项目

- venv (命名的环境)

```
ktlewis@ubuntu:~$ virtualenv --system-site-packages venv
```
- tensorflow (命名的环境)

```
ktlewis@ubuntu:~$ virtualenv --system-site-packages ~/tensorflow
```
>  $ virtualenv venv 
> 
> 目前版本默认不使用全局Python安装的库，即参数--no-site-packages。
> 如果要继承全局Python的库，则
> 
> $ virtualenv --system-site-packages venv

###  4、激活虚拟环境
使用前，需要先激活虚拟环境

```
ktlewis@ubuntu:~/tensorflow$ source bin/activate
```
### 5、使用pip安装 tensorflow

```
(tensorflow) ktlewis@ubuntu:~/tensorflow$ pip install tensorflow==1.4.1
```
> **注意** 墙内的朋友可使用 TUNA 的软件源镜像    Ubuntu 的软件源配置文件是 /etc/apt/sources.list

### 6、测试tensorflow环境
- 测试

```
>>> import tensorflow as tf
>>> hello = tf.constant('Hello ,TensorFlow!')
>>> sess = tf.Session()
>>> print sess.run(hello)
```
### 7、Terminal --> New Window (open新终端窗口)
- mkdir文件夹

```
ktlewis@ubuntu:~$ cd tensorflow/
ktlewis@ubuntu:~/tensorflow$ ls
bin  include  lib  local
ktlewis@ubuntu:~/tensorflow$ mkdir dl21
ktlewis@ubuntu:~/tensorflow$ ls
bin  dl21  include  lib  local
ktlewis@ubuntu:~/tensorflow$ cd dl21/
ktlewis@ubuntu:~/tensorflow/dl21$ mkdir chapter_1
ktlewis@ubuntu:~/tensorflow/dl21$ ls
chapter_1

```
### 8、准备资源 (程序代码+数据)
- 获取github资源链接，从而在ubuntu 使用wget获取所需代码文件 
```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ wget https://github.com/hzy46/Deep-Learning-21-Examples/raw/master/chapter_1/download.py
```

```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ ls
download.py
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ cat download.py 
```


```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ mkdir MNIST_data
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ ls
download.py  MNIST_data
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ cd MNIST_data/
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1/MNIST_data$ ls

```
- 下载随书相关文件
> https://github.com/hzy46/TensorFlow-Eager-Examples/tree/master/data/MNIST_data


```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1/MNIST_data$ wget https://github.com/hzy46/TensorFlow-Eager-Examples/raw/master/data/MNIST_data/t10k-images-idx3-ubyte.gz

ktlewis@ubuntu:~/tensorflow/dl21/chapter_1/MNIST_data$ wget https://github.com/hzy46/TensorFlow-Eager-Examples/raw/master/data/MNIST_data/t10k-labels-idx1-ubyte.gz

ktlewis@ubuntu:~/tensorflow/dl21/chapter_1/MNIST_data$ wget https://github.com/hzy46/TensorFlow-Eager-Examples/raw/master/data/MNIST_data/train-images-idx3-ubyte.gz

ktlewis@ubuntu:~/tensorflow/dl21/chapter_1/MNIST_data$ wget https://github.com/hzy46/TensorFlow-Eager-Examples/raw/master/data/MNIST_data/train-labels-idx1-ubyte.gz

```
> TensorFlow教程中有些直接import input_data就会报错ImportError: No module named input_data

### 解决方法
- 先安装[input_data.py](https://blog.csdn.net/u011068475/article/details/70338494)


```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ wget https://github.com/greedygrammar/tensorflow/raw/master/tensorflow/examples/tutorials/mnist/input_data.py
--2019-01-07 21:44:55--  https://github.com/greedygrammar/tensorflow/raw/master/tensorflow/examples/tutorials/mnist/input_data.py

```
- 注意 因在激活虚拟环境之后才安装的tensorflow，所以只有**激活了虚拟环境，才能使用对应的tensorflow版本**。
```
ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ source  ~/tensorflow/bin/activate
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ python input_data.py 
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ python download.py 
```

### 打印图片


```
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ wget https://github.com/hzy46/Deep-Learning-21-Examples/raw/master/chapter_1/save_pic.py

(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ python save_pic.py 
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$  pip install -U scipy
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ sudo pip install pillow
(tensorflow) ktlewis@ubuntu:~/tensorflow/dl21/chapter_1$ python save_pic.py 

```
> https://blog.csdn.net/silent56_th/article/details/79002152
> https://blog.csdn.net/huixingshao/article/details/80462282