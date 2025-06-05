# rockpi-4b-plus

# ROCK Pi 4B+ 安装 Armbian 后的操作步骤

## 1️⃣ 安装 NetworkManager，并修改 Netplan 配置

```bash
sudo apt install network-manager
sudo nano /etc/netplan/*.yaml
````

修改内容示例：

```yaml
network:
  version: 2
  renderer: NetworkManager
```

---

## 2️⃣ 使用 nmtui 关闭所有网卡的 IPv6 功能

```bash
sudo nmtui
```

* 进入 "Edit a connection"
* 编辑各网卡，设置 IPv6 Method 为 "disable"

---

## 3️⃣ 修改 bootenv，设置正确的 dtb 文件

```bash
vim /boot/armbianEnv.txt
```

修改 `fdtfile` 为：

```text
fdtfile=rk3399-rock-pi-4b-plus.dtb
```

实际路径示例，确认系统中存在:

```text
/usr/lib/linux-image-6.12.32-current-rockchip64/rockchip/rk3399-rock-pi-4b-plus.dtb
```

---

## 4️⃣ 修正 brcmfmac firmware 链接

```bash
cd /usr/lib/firmware/brcm/
sudo ln -s brcmfmac43456-sdio.radxa,rockpi4b.txt brcmfmac43456-sdio.radxa,rockpi4b-plus.txt
sudo ln -s brcmfmac43456-sdio.radxa,rockpi4b.bin brcmfmac43456-sdio.radxa,rockpi4b-plus.bin
```

---

## 5️⃣ 创建自定义 device tree overlay 限制 WiFi 频率

```bash
sudo mkdir -p /boot/overlay-user
sudo nano /boot/overlay-user/sdio-fix.dts
```

内容示例：

```dts
/dts-v1/;
/plugin/;

/ {
    compatible = "rockchip,rk3399";

    fragment@0 {
        target = <&sdio0>;
        __overlay__ {
            max-frequency = <100000000>; /* SDR50 */
            keep-power-in-suspend;
        };
    };
};
```

编译 overlay：

```bash
armbian-add-overlay /boot/overlay-user/sdio-fix.dts
```

检查dtbo是否生成：

```bash
ls /boot/overlay-user/sdio-fix.dtbo
```

确认 `/boot/armbianEnv.txt` 中是否包含：

```text
user_overlays=sdio-fix
```

---

## 6️⃣ 更新系统软件包

```bash
curl 'https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xdf00faf1c577104b50bf1d0093d6889f9f0e78d5' |gpg --dearmor > /usr/share/keyrings/armbian.gpg
sudo apt update
sudo apt upgrade -y
```

---

# 📚 参考链接

- **4B+ 的 DTS 描述** ： [https://github.com/radxa/kernel/blob/linux-6.1-stan-rkr5.1/arch/arm64/boot/dts/rockchip/rk3399-rock-pi-4b-plus.dts](https://github.com/radxa/kernel/blob/linux-6.1-stan-rkr5.1/arch/arm64/boot/dts/rockchip/rk3399-rock-pi-4b-plus.dts)

- **Armbian 的 ROCK Pi 4 页面** ： [https://www.armbian.com/rockpi4/](https://www.armbian.com/rockpi4/)

- **Armbian 社区支持镜像发布地址** ： [https://github.com/armbian/community/releases](https://github.com/armbian/community/releases)

- **刷机用最新的 4B Armbian 镜像** ： [Armbian_community_25.8.0-trunk.90_Rockpi-4b_bookworm_current_6.12.31_minimal.img.xz](https://github.com/armbian/community/releases/download/25.8.0-trunk.90/Armbian_community_25.8.0-trunk.90_Rockpi-4b_bookworm_current_6.12.31_minimal.img.xz)

- **兜底镜像（wlan0 挂了可用），刷完 wlan0 可恢复** ： [rockpi4b-ubuntu-bionic-minimal-20191127_1942-gpt.img.gz](https://dl.radxa.com/rockpi/images/ubuntu/rockpi4b-ubuntu-bionic-minimal-20191127_1942-gpt.img.gz)

- **radax刷机文档(loader使用rk3399_loader_v1.27.126.bin)** : [https://wiki.radxa.com/Rockpi4/dev/usb-install](https://wiki.radxa.com/Rockpi4/dev/usb-install)
