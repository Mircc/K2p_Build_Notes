# 前言
在hub.docker.com上已经有很多n1可用的openwrt镜像了，但是其功能可能并不是那么符合自己的需要，这里可以自己制作一个docker的openwrt镜像。

# 过程
## 安装docker
生成docker镜像必不可少的就是安装好docker

`curl -fsSL https://get.docker.com | sudo bash`
## 编译openwrt
首先是从源码编译出一个openwrt镜像。
这里可以参考：[编译k2p的openwrt固件](https://zorz.cc/post/k2p-compile-openwrt.html "编译k2p的openwrt固件")
但是目标设备选择不同。
这里的选择是

`Target选 "QEMU ARM Virtual Machine" > "ARMv8 multiplatform"
Target Images 要选上tar.gz`
## 一键脚本
这里感谢恩山的@flippy，他制作了一键脚本方便生成docker镜像：[make_opwrt_docker_img.tar.gz](https://github.com/Mircc/K2p_Build_Notes/releases/tag/N1_openwrt_make")
下载好脚本之后
解压脚本，可以将脚本放置到你喜欢的目录，将之前生成的tar.gz文件放到脚本同一目录。
创建存放生成的镜像打包文件的目录

`sudo mkdir -p /opt/imgs/docker`
开始生成镜像，在脚本存在目录执行

`sudo bash build.sh tag
`tag就是docker镜像的版本号，例如r9.10.24，tag可以为空，则其为默认值：latest
脚本用到了pigz如果没有这个工具，需要安装

`apt-get install pigz`
可以在/opt/imgs/docker目录查看生成好的镜像文件

## 导入镜像
将生成好的镜像文件传输到N1，运行以下命令导入
`docker load --input 镜像文件
`
导入成功之后，可以参考这篇文章进行使用：N1在armbian的docker中使用openwrt

## 脚本说明
```bash
#!/bin/bash
TAG=latest    # 版本号，默认是latest
if [ ! -z "$1" ];then
        TAG=$1
fi
 
TMPDIR=openwrt_rootfs
OUTDIR=/opt/imgs/docker    # 本地镜像保存路径，可以自己修改，但目录要提前建好
IMG_NAME=unifreq/openwrt-aarch64   # 镜像名，可以自己修改
 
[ -d "$TMPDIR" ] && rm -rf "$TMPDIR"
 
mkdir -p "$TMPDIR"  && \
gzip -dc openwrt-armvirt-64-default-rootfs.tar.gz | ( cd "$TMPDIR" && tar xf - ) && \
cp -f patches/rc.local "$TMPDIR/etc/" && \
cp -f patches/cpustat "$TMPDIR/usr/bin/" && \
chmod 755 "$TMPDIR/usr/bin/cpustat" && \
cat patches/luci-admin-status-index-html.patch | (cd "$TMPDIR/usr/lib/lua/luci/view/admin_status/" && patch -p0) && \
sed -e "s/net.nf_conntrack_max net.ipv4.netfilter.ip_conntrack_max/net.netfilter.nf_conntrack_max net.nf_conntrack_max net.ipv4.netfilter.ip_conntrack_max \| head -n 1/" -i "$TMPDIR/usr/lib/lua/luci/view/admin_status/index.htm" && \
rm -f "$TMPDIR/etc/bench.log" && \
echo "17 3 * * * /etc/coremark.sh" >> "$TMPDIR/etc/crontabs/root" && \
(cd "$TMPDIR" && tar cf ../openwrt-armvirt-64-default-rootfs-patched.tar .) && \
rm -f DockerImg-OpenwrtArm64-${TAG}.gz && \
docker build -t ${IMG_NAME}:${TAG} . && \
rm -f  openwrt-armvirt-64-default-rootfs-patched.tar && \
rm -rf "$TMPDIR" && \
docker save ${IMG_NAME}:${TAG} | pigz -9 > $OUTDIR/docker-img-openwrt-aarch64-${TAG}.gz
```
参考链接：
[N1及贝壳云Armbian 5.98(加强版)， 内核5.3.x， 及 Docker Openwrt](https://www.right.com.cn/forum/thread-958173-1-1.html "N1及贝壳云Armbian 5.98(加强版)， 内核5.3.x， 及 Docker Openwrt")
[斐讯N1 / 贝壳云 一键制作OpenWrt镜像脚本](https://github.com/tuanqing/mknop "斐讯N1 / 贝壳云 一键制作OpenWrt镜像脚本")
[制作n1的docker版openwrt镜像](https://zorz.cc/post/n1-compile-openwrt-docker-image.html "制作n1的docker版openwrt镜像")
