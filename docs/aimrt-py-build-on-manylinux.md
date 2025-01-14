# AimRT-Py 在 manylinux 上的构建过程

## 一、背景

现需要构建在各种 linux 发行版上通用的 AimRT-Py 包，需要一个足够低版本的 glibc 的 linux 发行版，一方面解决用户 glibc 版本低于 Ubuntu 22.04 上默认的 2.30 的问题，另一方面方便制作可以上传到 PyPI 上的二进制包。

[Manylinux](https://github.com/pypa/manylinux) 是针对二进制 python 包构建的专用镜像，在其之上构建出的 python 包，可以保证在各种 linux 发行版上通用，专门用于制作可以上传到 PyPI 上的二进制包。

## 二、过程记录

基本环境准备，拉取镜像、[换国内源](https://www.cnblogs.com/sysin/p/18256193)

```bash
docker pull quay.io/pypa/manylinux_2_28_x86_64:latest
docker run -it quay.io/pypa/manylinux_2_28_x86_64:latest /bin/bash


sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^# baseurl=https://repo.almalinux.org|baseurl=https://mirrors.aliyun.com|g' \
    -i.bak \
    /etc/yum.repos.d/almalinux*.repo
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https://download.example/pub|baseurl=https://mirrors.aliyun.com|g' \
    -e 's|^#baseurl=https://download.fedoraproject.org/pub|baseurl=https://mirrors.aliyun.com|g' \
    -i.bak \
    /etc/yum.repos.d/epel*.repo


export ORIGIN_PATH=$PATH
export python_version="cp310-cp310"
export PATH="/opt/python/${python_version}/bin:$PATH"

python3 -m pip install --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```



二进制安装 Ros2 Humble 可以参考

<https://docs.ros.org/en/humble/Installation/RHEL-Install-RPMs.html>



还需要额外做一些修复项：

1. 编译 AimRT ros2_plugin 过程中需要安装一些缺少的 python 包

    ```bash
    pip install empy==3.3.4 catkin_pkg numpy lark
    ```

2. 编译 iceoryx 插件需要安装 libatomic-devel 和 libacl-devel，编译 opentelemetry 插件需要安装 libcurl-devel

    ```bash
    dnf install gcc-toolset-14-libatomic-devel libacl-devel libcurl-devel
    ```

3. 此外针对 AimRT 仓库本身，还需要修改一下 pybind11 的头文件包含问题（gcc14 才出现）

    <https://github.com/pybind/pybind11/issues/5206>


构建命令

```bash
cd /home
rm -rf aimrt
git clone https://github.com/AimRT/AimRT.git aimrt --depth=1
cd aimrt

source /opt/ros/humble/setup.bash
./build.sh \
      -DAIMRT_BUILD_EXAMPLES=OFF \
      -DAIMRT_BUILD_DOCUMENT=OFF \
      -DAIMRT_BUILD_MQTT_PLUGIN=OFF
```

最后对构建出的 wheel 包执行 `auditwheel repair` 命令，修复 wheel 包的 ABI 兼容性问题。

```bash
auditwheel repair build/aimrt_py_pkg/dist/aimrt_py-*.whl
```

最终修复后的 wheel 包位于 `/home/aimrt/wheelhouse` 目录下，其 ABI tag 为 `manylinux_2_28_x86_64`。
