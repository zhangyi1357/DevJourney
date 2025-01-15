# AimRT-Py 在 manylinux 上的构建过程

## 一、背景

现需要构建在各种 linux 发行版上**通用**的 AimRT-Py 包，需要一个足够低版本的 glibc 的 linux 发行版，一方面解决用户 glibc 版本低于 Ubuntu 22.04 上默认的 2.30 的问题，另一方面方便制作可以上传到 PyPI 上的二进制包。

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

[https://docs.ros.org/en/humble/Installation/RHEL-Install-RPMs.html](https://docs.ros.org/en/humble/Installation/RHEL-Install-RPMs.html)



还需要额外做一些修复项：

1. 编译 AimRT ros2_plugin 过程中需要安装一些缺少的 python 包

   ```bash
   pip install empy==3.3.4 catkin_pkg numpy lark
   ```

2. 编译 iceoryx 插件需要安装 libatomic-devel 和 libacl-devel，编译 opentelemetry 插件需要安装 libcurl-devel

   ```bash
   dnf install gcc-toolset-14-libatomic-devel libacl-devel libcurl-devel
   ```

3. 编译 zenoh 插件需要安装 rust，而默认源安装较慢，建议使用国内源

   ```bash
   # 可以把以下两行添加到 ~/.bashrc 等文件，使之成为全局环境变量，这样后续所有 rustup 或 cargo 命令均会使用此环境变量
   export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
   export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup

   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

   以上是加速 rust 本身的安装，如果要加速 rust 包的下载，可以替换 crates.io 源，更新 ~/.cargo/config 文件，内容如下

   ```bash
   [source.crates-io]
   registry = "https://github.com/rust-lang/crates.io-index"

   # 替换成你偏好的镜像源
   replace-with = 'sjtu'
   #replace-with = 'ustc'

   # 清华大学
   [source.tuna]
   registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

   # 中国科学技术大学
   [source.ustc]
   registry = "git://mirrors.ustc.edu.cn/crates.io-index"

   # 上海交通大学
   [source.sjtu]
   registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"

   # rustcc社区
   [source.rustcc]
   registry = "git://crates.rustcc.cn/crates.io-index"
   ```

4. 此外针对 AimRT 仓库本身，还需要修改一下 pybind11 的头文件包含问题（gcc14 才出现）

   [https://github.com/pybind/pybind11/issues/5206](https://github.com/pybind/pybind11/issues/5206)


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



## 三、结果

可以在 manylinux_2_28 上完整编译 AimRT，但是最终编出来的包中 ros2 相关的 .so 依赖一些在 manylinux 镜像上才有的动态库，例如 ros2  自身的一些动态库，还有其系统自带的 python3.6，导致其无法在 Ubuntu 22.04 上正常使用。

此外一些其他的插件包也有一些符号依赖的问题，使用 [auditwheel-symbols](https://github.com/messense/auditwheel-symbols) 工具检查结果如下：

````
# auditwheel-symbols -m 2_28 build/aimrt_py_pkg/dist/aimrt_py-0.10.0-cp310-cp310-linux_x86_64.whl
aimrt_py/aimrt_python_runtime.cpython-310-x86_64-linux-gnu.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_echo_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_grpc_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_iceoryx_plugin.so is not manylinux_2_28 compliant because it links the following forbidden libraries:
libacl.so.1
libatomic.so.1
aimrt_py/libaimrt_log_control_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_mqtt_plugin.so is not manylinux_2_28 compliant because it links the following forbidden libraries:
libanl.so.1
libssl.so.1.1
libcrypto.so.1.1
aimrt_py/libaimrt_net_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_opentelemetry_plugin.so is not manylinux_2_28 compliant because it links the following forbidden libraries:
libcurl.so.4
aimrt_py/libaimrt_parameter_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_proxy_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_record_playback_plugin.so is manylinux_2_28 compliant.
aimrt_py/libaimrt_time_manipulator_plugin.so is manylinux_2_28 compliant.
````

这与上述安装的内容有一定的对应关系。

如果将这样的包上传 PyPI，其中需要去除至少 ros2、mqtt 相关的功能和插件，插件可以容易地下载，但是 ros2 相关功能想要通过额外的方式提供具有一定的困难，最终还是需要用户在指定的 Ubuntu 22.04 平台上运行，又需要额外下载内容，范式发散，得不偿失。

结论：

1. Wheel 包中至少有 ROS2 接口、ROS2 插件、mqtt 插件无法完整得到跨发行版二进制；
2. 将部分功能二进制上 PyPI 会导致用户使用时的困惑，增大维护难度；
3. 暂时不上 PyPI；
4. Conda 使用问题通过文档说明进行 workaround（将 conda 中的 libstdc++ 软链接到系统的 libstdc++ 即可）。

