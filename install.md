# Canoe v1.4 安装记录 (HPC: 中科曙光)

注: 如果有文档解决不了的问题，请咨询chatgpt（不要咨询deepseek）


# 基本环境

本次安装环境：
- 系统: Linux (HPC集群)
- 编译器: GCC 7.3.1 (devtoolset-7)
- Python: 3.8.10 (module提供)
- CMake: 需要 ≥ 3.20 (系统默认过低)

# 源代码下载

网址:` https://github.com/chengcli/canoe/releases/tag/v1.4`

代码储存在最后Assets中，下载Source conde (zip)

将代码上传到服务器中并解压: `unzip canoe-1.4.zip`

进入解压的文件夹: `cd canoe-1.4.zip`

开始照着README安装 -> (如果失败了就看一下文档)

# 第一步遇到问题

## 服务器sudo权限不够

报错: 安装文档中要求输入`sudo yum -y install $(cat packages_centos.txt)`但是用户没有sudo权限

解决: 检查可用模块并加载

`packages_centos.txt`要求安装这些库:

```bash
clang-tools-extra (x)

cmake (自己安装)

nco (module)

ncview (module)

netcdf-devel (module)

boost-devel (x)

eigen3-devel (自己安装)

glog-devel （x）

fftw-devel (有)

blas-devel (有)

lapack-devel (?)
```

### 服务器中已有的module

加载服务器已有的module:

```bash
module load compiler/devtoolset/7.3.1

module load python/3.8.10

module load mathlib/netcdf/c4.9.2_f4.6.1-gcc731

module load mathlib/nco/4.6.7/intel

module load mathlib/ncview/2.1.7/intel
```

验证:

```bash
which gcc

which python

which nc-config
```


### 我手动安装的

#### CMake

服务器中cmake版本过低，需要手动安装，执行命令行:

```bash
cd ~/opt

wget https://github.com/Kitware/CMake/releases/download/v3.29.6/cmake-3.29.6-linux-x86_64.tar.gz

tar -xzf cmake-3.29.6-linux-x86_64.tar.gz
```

加入path:

```bash
export PATH=$HOME/opt/cmake-3.29.6-linux-x86_64/bin:$PATH
```

验证:

```bash
cmake --version
```

#### Eigen3

不记得为什么这个库是手动安装的了，执行命令行:

```bash
cd ~/opt

wget https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.tar.gz

tar -xzf eigen-3.4.0.tar.gz
```

#### 注意

其他的没有安装的库(boost-devel, glog-devel, clang-tools-extra)就暂时没管，需要的话我再安装（其余的库不用canoe是可以安装的）

这一部分结束了，照着README继续安装 -> (如果失败了就看一下文档)

# 第二部遇到问题

## 配置必要的python环境

创造`pyenv`虚拟环境需要先load python模块: 

```bash
module load python/3.8.10
```

执行命令行:

```bash
python -m venv pyenv

source pyenv/bin/activate

pip install -r requirements.txt
```

这一部分结束了，照着README继续安装 -> (如果失败了就看一下文档)

# 第三步遇到问题

## 编译canoe

执行命令行，并注意使用example配置:

```bash
cd ~/canoe-1.4

mkdir build

cd build

cmake .. -C ../cmake/examples/rcemip.cmake \
  -DEIGEN3_INCLUDE_DIR=$HOME/opt/eigen-3.4.0

make -j4
```

## Dropbox下载失败

如一些辐射数据，李成老师会放在dropbox中，如果无法访问，需要在本地下载->上传至服务器data文件夹中->手动解压

```bash
tar -xzf *.tar.gz
```

## wget报错

```bash
wget: unrecognized option '--show-progress'
```

解决:

```bash
cd ~/canoe-1.4

sed -i 's/--show-progress//g' data/fetch_*.sh
```

# 创建环境脚本（推荐）

创建`~/canoe_env.sh`

```bash
module load compiler/devtoolset/7.3.1

module load python/3.8.10

module load mathlib/netcdf/c4.9.2_f4.6.1-gcc731

module load mathlib/nco/4.6.7/intel

module load mathlib/ncview/2.1.7/intel

source ~/canoe-1.4/pyenv/bin/activate

export PATH=$HOME/opt/cmake-3.29.6-linux-x86_64/bin:$PATH

export CANOE_HOME=$HOME/canoe-1.4

export PATH=$CANOE_HOME/build/bin:$PATH

export PYTHONPATH=$CANOE_HOME/build/python:$PYTHONPATH

echo "[Canoe] environment loaded"
```

在~/.bashrc中添加:

```bash
alias canoe='source ~/canoe_env.sh'
```

使用`canoe`来加载canoe所需要的环境