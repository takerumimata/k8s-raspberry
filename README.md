# k8sのラズベリーパイ包み
本リポジトリはRaspberry Pi 3 Model B+を３台利用したk8sクラスターの構築を行った際の手順をメモしておくリポジトリ。

# OS setup
```
OS: Hypriot Version 1.12.3
user: pirate
password: hypriot
```
以下のリンクより、imgをダウンロードし、micro sdに書き込む
[hypriotイメージのダウンロードリンク](https://blog.hypriot.com/downloads/)

```zsh
// unzip img
$ unzip hypriotos-rpi-v****.img.zip

// SDカードが認識されているか確認
$ diskutil list
~~中略~~
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.1 GB    disk2
   1:             Windows_FAT_32                         31.1 GB    disk2s1

// SDカードをアンマウント
$ diskutil unmountdisk /dev/disk2
Unmount of all volumes on disk2 was successful

// imageをSDに書き込む
$ sudo dd if=hypriotos-rpi-v1.10.0.img of=/dev/rdisk2 bs=1m
Password:
1000+0 records in
1000+0 records out
1048576000 bytes transferred in 60.963957 secs (17199933 bytes/sec)
```

ここまでできたら、sdをぶっ刺すだけで、sshする準備が整う。arp-scanでラズパイを探す
```zsh
sudo arp-scan -l --interface en0
Password:
Interface: en0, type: EN10MB, MAC: f0:18:98:1b:f5:f3, IPv4: 192.168.11.4
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.11.1	58:27:8c:7c:11:b8	(Unknown)
192.168.11.16	b8:27:eb:c0:fc:f7	Raspberry Pi Foundation
192.168.11.5	4c:ef:c0:40:5e:66	Amazon Technologies Inc.
192.168.11.7	40:49:0f:dd:ee:93	Hon Hai Precision Ind. Co.,Ltd.
192.168.11.17	b8:27:eb:31:01:39	Raspberry Pi Foundation
192.168.11.18	b8:27:eb:05:a6:ef	Raspberry Pi Foundation
192.168.11.19	b8:27:eb:ba:71:52	Raspberry Pi Foundation

518 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 1.851 seconds (138.30 hosts/sec). 7 responded
```

パッケージの更新
```zsh
$ sudo apt-get update \
  && sudo apt-get -y dist-upgrade \
  && sudo apt-get -y autoremove \
  && sudo apt-get autoclean
```
vimを利用する
```
$ echo "set nocompatible" > ~/.vimrc
$ sudo -i
# echo "set nocompatible" > ~/.vimrc
```

static ipの割り当てに関しては、dhcpcd.confに書き込んだ内容がなぜか適用されない（issue1を参照、安定してないっぽい？）ので省略。

# k8sのinstall

[公式のinstall Documents](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)にしたがってインストールすれば良い
```zsh
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
# 参考
[ラズパイでk8s構築する](https://esakat.github.io/esakat-blog/posts/raspberrypi-k8s-setup/)
[ラズパイでKubernetesクラスタを構築する](https://qiita.com/sotoiwa/items/e350579d4c81c4a65260)
