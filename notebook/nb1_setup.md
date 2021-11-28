
# Arudino IDEのインストール
1. Arduino IDE のダウンロード https://www.arduino.cc/en/software
	- 2021/11/17時点では 1.18.16
	- windows, mac, linux 対応版が存在する
	- jetson nano 2GB では、ARM64 をインストールすること
2. IDEの起動
	- デスクトップのショートカットで起動するか、インストールした場所で ./arduino で起動する

## Jetson nano にインストール
1. 解凍、セットアップスクリプトの実行

```shell
tar xvf arduino-1.8.16-linuxaarc64.tar.xz                                 
mv arduino-1.8.16 ~/.local/                                                      
cd ~/.local/arduino-1.8.16                                                           
source arduino-linux-setup.sh $USER                                            
```

2. 再起動
3. install.sh の実行

```shell
cd arduino-1.8.16/                                                          
sudo source install.sh                                                 
```

これで起動できる。X を飛ばしていれば、mac から jetson にssh接続して、画面をもってくることができる（ちょっと遅いけど）。なので、GUIではなくCUIからスケッチを書き込めるように設定を追加で行う。

## CUIからのスケッチの書き込み
platformio のインストール

```shell
sudo apt install arduino
sudo apt install python3-pip（platformioはpython3.6以上が必要）
sudo pip3 install platformio
```

以上で platformio の設定が終わるはずで、platformio のコマンドは無いのだが（なぜ？）、pio という相当するコマンドはインストールされている。

```shell
which pio
/usr/local/bin/pio
```

init を打つと、src, lib, include ディレクトリが自動で生成される。また、platformio.ini ファイルも生成され、 このファイルはプロジェクトの設定ファイルである。




```shell
ktakeda@ktakeda:~/workspace/test$ pio init --board=uno
***********************************************************************************************************************
If you like PlatformIO, please:
- follow us on Twitter to stay up-to-date on the latest project news > https://twitter.com/PlatformIO_Org
- star it on GitHub > https://github.com/platformio/platformio
- try PlatformIO IDE for embedded development > https://platformio.org/platformio-ide
***********************************************************************************************************************


The current working directory /home/ktakeda/workspace/test will be used for the project.

The next files/directories have been created in /home/ktakeda/workspace/test
include - Put project header files here
lib - Put here project specific (private) libraries
src - Put project source files here
platformio.ini - Project Configuration File

Project has been successfully initialized! Useful commands:
`pio run` - process/build project from the current directory
`pio run --target upload` or `pio run -t upload` - upload firmware to a target
`pio run --target clean` - clean project (remove compiled files)
`pio run --help` - additional information
```


## Jetson と Arduion を接続する

arduino と USB ケーブルで接続したあとで、lsusb コマンドで接続されているかを確認する。以下の例では正しく認識されていることが分かり、BUS001 Device 006 で認識されている。

```shell
ktakeda@ktakeda:~/workspace/test$ lsusb 
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 2341:0043 Arduino SA Uno R3 (CDC ACM)
Bus 001 Device 003: ID 1a40:0801 Terminus Technology Inc. 
Bus 001 Device 002: ID 0bda:8179 Realtek Semiconductor Corp. RTL8188EUS 802.11n Wireless Network Adapter
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Arduino にスケッチを書き込むためには、対応するデバイスファイルを知る必要がある。そこで`dmesg`コマンドを実行して、情報を抜き出す。以下から `/dev/ttyACM0` であることが分かる。

```shell
[701533.603928] usb 1-3.1: new full-speed USB device number 6 using tegra-xusb
[701533.628062] usb 1-3.1: New USB device found, idVendor=2341, idProduct=0043
[701533.628067] usb 1-3.1: New USB device strings: Mfr=1, Product=2, SerialNumber=220
[701533.628070] usb 1-3.1: Manufacturer: Arduino (www.arduino.cc)
[701533.628073] usb 1-3.1: SerialNumber: 95038303231351800191
[701533.733202] cdc_acm 1-3.1:1.0: ttyACM0: USB ACM device
[701533.733881] usbcore: registered new interface driver cdc_acm
[701533.733885] cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
```

調べたデバイスファイルを `platformio.ini` ファイルの中に、`upload_port = /dev/...` として追記する。


## Lチカする

設定を終えた jetson, arduino を用いて早速Lチカを行ってみる。作成するスケッチは `src` 以下に作成する。
作成するファイルは

```shell
pio run --target=upload
```

でボードに書き込むことができる（一番初めには色々とライブラリのインストールなどが自動で行われる模様）。



# C++ で開発する

`src/` 以下に通常のファイルを配置し、`#include <Arduino.h>` をインクルードすることでC++で開発を進めることができる。platformio でビルドだけを行いたい場合は、`-t upload`などを付けず `pio run` の実行でビルドだけ行うことができる。