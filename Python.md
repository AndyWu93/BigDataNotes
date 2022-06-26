# 部署

- 在安装 Python之前，需要先安装编译时用到的依赖

```shell
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install libffi-devel -y
```

- 下载

```shell
wget https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tar.xz
```

- 解压

```shell
tar -xvJf  Python-3.6.0.tar.xz

# 创建编译安装的目录
mkdir /usr/local/python3
```

- 进入刚才解压后得到的文件夹中，然后进行编译

```shell
cd Python-3.6.0
./configure --prefix=/usr/local/python3
make && make install
```

- 为了方便我们在命令行执行 python 和 pip，这里需要为 Python创建**软链接**

```shell
ln -s /usr/local/python3/bin/python3 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3
```

- check

```shell
python3 --version
pip3 --version
```







# 备注

## uwsgi

- https://mp.weixin.qq.com/s?__biz=Mzg5NzIyMzkzNw==&mid=2247484134&idx=1&sn=f2eb075958179f26ad9328eb82fa918a&chksm=c0745e88f703d79eae04512cc47bc044b256fc9cdc0bf0e4e0cb6ec69151fff18fca4d1e0ca2&mpshare=1&scene=23&srcid=0616GV9s02XkNN6STuQqXXNR&sharer_sharetime=1623824772502&sharer_shareid=c1e370872ac83c76a1f6355a358dc03a%23rd