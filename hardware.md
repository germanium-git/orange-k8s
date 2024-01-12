# Hardware

![orangepi](https://raw.githubusercontent.com/germanium-git/orange-k8s/main/pictures/orangepi.png)

## Bill of Materials

### Main components

Master Node - 1pc [Orange Pi 5 8GB Rockchip RK3588S 8 Core 64 Bit Single Board Computer, 2.4GHz Frequency Open Source Development Board Mini PC Desktop Run Orange Pi OS, Android12, Debian11 (Pi 5 8GB)](https://www.amazon.de/gp/product/B0BVJPM1SP/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)

Worker nodes - 2pcs [Orange Pi 5 16GB RK3588S,PCIE Module External WiFi+BT,SSD Gigabit Ethernet Single Board Computer,Run Android Debian OS](https://www.aliexpress.com/item/1005004957723357.html)

SSD - 3pcs [Samsung SSD 256GB PM991 M.2 2242 42 mm PCIe 3.0 x4 NVMe MZALQ256HAJD MZ-ALQ2560 Solid State Drive for Lenovo Dell HP Acer Asus Others Solid State Drive](https://www.amazon.de/gp/product/B08K79T9G5/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1)

Case - [Geekworm Orange Pi 5/5B Cluster Case with Fan Kit, Acrylic Stack Shell / Enclosure for Orange Pi 5/5B](https://www.aliexpress.com/item/1005005578980759.html?spm=a2g0o.order_list.order_list_main.80.5f2e1802RH040c)

Power Supply - [ssouwao 120 W GaN 6-Port Charger, Power Supply with 3 USB-C and 3 USB-A, Quick Charging Station, Multiple Charging Plug, Laptop Charger for MacBook, iPhone, iPad, Galaxy](https://www.amazon.de/gp/product/B0BRR64F64/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)


### Additional items

M.2 NVME Enclosure - [M.2 NVME Enclosure, Beikell Toolless 10Gbps M.2 SSD Enclosure USB 3.2 Gen 2 NVMe to USB Adapter for 2230/2242/2260/2280 M.2 NVMe/SATA SSD from M-Key/M+B Key with USB C to C and USB-A to C Cable](https://www.amazon.de/gp/product/B0BGS3NZ4C/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)


## Pictures

![orangepi_cluster1](https://raw.githubusercontent.com/germanium-git/orange-k8s/main/pictures/orangepi_cluster1.jpeg)

![orangepi_cluster2](https://raw.githubusercontent.com/germanium-git/orange-k8s/main/pictures/orangepi_cluster2.jpeg)


## HLD Installation

1. Download [Armbian (CLI version)](https://www.armbian.com/orangepi-5/) image
2. Untar the content
3. Flash Armbian OS image to SD card with [balenaEtcher](https://etcher.balena.io/)
4. Flash OS image to NVMe SSD with [balenaEtcher](https://etcher.balena.io/)
5. Have the Orange Pi boot up.
6. Make Orange Pi 5: Boot from M.2 NVMe SSD for Armbian
    - Follow the [video](https://www.youtube.com/watch?v=dncGA3qg_8w&ab_channel=Ace1000ks1975) on YT as a reference.
    - sudo armbian-config
    - System (System and security settings) > Install (Install to/update boot loader ) > 7 Install/Update the booytloader on MTD Flash > Continue? Yes > wait a few minutes > Back > Exit
    - Shut down the computer, remove the SD card and turn it on again.
    - Go through the initial settings again.
7. Install K8s with kubeadm on the master node.