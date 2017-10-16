---
layout: post
title: mpy4py 环境配置
categories: environment
description: 在 Linux 环境安装 openMPI 和 mpy4py 的方法
keywords: linux, mpi4py
---

## 安装 Open MPI
OpenMPI 官网：[openmpi](www.open-mpi.org)  
目前最新版本为 3.0.0
```
wget https://www.open-mpi.org/software/ompi/v3.0/downloads/openmpi-3.0.0.tar.gz
tar -xzvf openmpi-3.0.0.tar.gz
cd openmpi-3.0.0
./configure --prefix=/安装目录/openmpi-3.0.0
make all install
```
安装后添加环境变量
```
export PATH=/安装目录/openmpi-3.0.0/bin:$PATH
export LD_LIBRARY_PATH=/安装目录/openmpi-3.0.0/lib:$LD_LIBRARY_PATH
```

## 安装 mpi4py
下载 mpi4py：[mpi4py](https://pypi.python.org/pypi/mpi4py)
```
wget https://pypi.python.org/packages/ee/b8/f443e1de0b6495479fc73c5863b7b5272a4ece5122e3589db6cd3bb57eeb/mpi4py-2.0.0.tar.gz
tar -xzvf mpi4py-2.0.0.tar.gz
cd mpi4py-2.0.0
vim mpi.cfg
```
修改 mpi_dir 为 openmpi 的安装目录
```
# Open MPI example
# ----------------
[openmpi]
mpi_dir              = /安装目录/openmpi-3.0.0
mpicc                = %(mpi_dir)s/bin/mpicc
mpicxx               = %(mpi_dir)s/bin/mpicxx
#include_dirs         = %(mpi_dir)s/include
#libraries            = mpi
library_dirs         = %(mpi_dir)s/lib
runtime_library_dirs = %(library_dirs)s
```
继续安装 mpi4py
```
python setup.py install
```
删除方法
```
python setup.py install --record log
cat log | xargs rm -rf
```

## 验证安装是否成功
在 python 中执行
```
import mpi4py.MPI
```
没有报错说明安装成功