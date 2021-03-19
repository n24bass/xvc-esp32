# Xilinx Virtual Cable Server for ESP32

これは下記の [cinimlさんの実装](https://github.com/ciniml/xvc-esp32)にシリアルアダプタ機能を追加したものです。
使用ピンの定義を多少変更している他、スタティック IP アドレス指定を追加しています。
なお、現状では xvc として接続されていないとシリアルアダプタが機能しません。今後改善の予定です。

## 概要

Xilinx製FPGAへの書き込みを行うためのプロトコル (XVC : Xilinx Virtual Cable) のESP32向け実装です。

Derek Mulcahy氏の Raspberry Pi向け実装(https://github.com/derekmulcahy/xvcpi) をESP32向けに移植しています。

ESP32を対象FPGAのJTAGピン (TDI, TDO, TMS, TCK) に接続することにより、WiFi経由でXilinx製のツール (Vivadoなど) からFPGAのJTAGポートにアクセスすることができます。

![M5Atom](picture.jpg)

写真の例では、M5Stack製のESP32モジュール [M5Atom Lite](https://docs.m5stack.com/#/en/core/atom_lite) とZynq XC7Z010を接続しています。

この状態で、VivadoのHardware Managerから `Add Virtual Cable` でVirtual Cableとして追加して接続すると、通常のJTAGアダプタと同様にILAの波形観測などを行えます。

![ILA](vivado_ila.png)

## 使い方

`xvc-esp32.ino` を使いたいESP32ボード向けにArduino IDEでビルドして書き込むだけです。

その際、現状のスケッチではWiFiの接続先をコードに埋め込んでいますので、適宜修正してください。

```c++
static const char* MY_SSID = "ssid";
static const char* MY_PASSPHRASE = "wifi_passphrase";
```

また、現状は M5Atom用のピン定義になっていますので、使いたいESP32ボード向けにピン定義を変更してください。

```
/* GPIO numbers for each signal. Negative values are invalid */
/* Note that currently only supports GPIOs below 32 to improve performance */
static constexpr const int tms_gpio = 22;
static constexpr const int tck_gpio = 19;
static constexpr const int tdo_gpio = 21;
static constexpr const int tdi_gpio = 25;
```

コメントにあるように、現状は高速化のためにGPIO0～31までのみ使用可能です。

## hw_server での使用例

Vivado では HARDWARE MANAGER から hw_manager/xvc 接続の設定ができますが、Vitis では hw_manager 指定まででその先の xvc 接続設定はできません。
そのため、下記の様に hw_manager の起動オプションで接続先となる xvc を指定して別に立ち上げておき、Vitis の Debug Configuration ではその hw_manager への接続を指定することで xvc が使えるようになります。hw_server は Vitis とは別のホストで動かしても構いません。

```
host:/tools/Xilinx/Vitis/2020.02/bin$ ./hw_server -e "set auto-open-servers xilinx-xvc:192.168.1.41:2542"
```

## ライセンス

大本の AVNET による実装が CC0 1.0 だったので、Derek Mulcahy氏の Raspberry Pi向け実装も CC0 1.0となっています。

従って、本実装も (日本国内でのCC0の有効性はともかく) CC0 1.0 ということにします。