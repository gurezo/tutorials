# CHIRIMEN for Raspberry Pi 3 チュートリアル 3. I2C　応用編（その他のセンサー）

# 概要

CHIRIMEN for Raspberry Pi 3 を使ったプログラミングを通じて、Web I2C API の使い方を学びます。 

前回は温度センサーを使いながらWeb I2C APIの基本的な利用方法を学びました。今回は温度センサー以外のI2Cセンサーの使い方を見ていきましょう。

## (※1) CHIRIMEN for Raspberry Pi 3とは
Raspberry Pi 3（以下「Raspi3」）上に構築したIoTプログラミング環境です。

[Web GPIO API (Draft)](https://rawgit.com/browserobo/WebGPIO/master/index.html)や、[Web I2C API (Draft)](https://rawgit.com/browserobo/WebI2C/master/index.html)といったAPIを活用したプログラミングにより、WebアプリからRaspi3に接続した電子パーツを直接制御することができます。 

CHIRIMEN Open Hardware コミュニティにより開発が進められています。

## 前回までのおさらい

本チュートリアルを進める前に前回までのチュートリアルを進めておいてください。

* [CHIRIMEN for Raspberry Pi 3 Hello World](section0.md)
* [CHIRIMEN for Raspberry Pi 3 チュートリアル 1. GPIO編](section1.md)
* [CHIRIMEN for Raspberry Pi 3 チュートリアル 2. I2C　基本編（ADT7410温度センサー）](section2.md)

前回までのチュートリアルで学んだことは下記のとおりです。


* CHIRIMEN for Raspberry Pi 3 では、各種exampleが `~/Desktop/gc/`配下においてある。配線図も一緒に置いてある
* CHIRIMEN for Raspberry Pi 3 で利用可能なGPIO Port番号と位置は壁紙を見よう
* CHIRIMEN for Raspberry Pi 3 ではWebアプリからのGPIOの制御にはWeb GPIO API を利用する。GPIOポートは「出力モード」に設定することでLEDのON/OFFなどが行える。また「入力モード」にすることで、GPIOポートの状態を読み取ることができる
* [async function](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function) を利用すると複数ポートの非同期コードがすっきり書ける
* CHIRIMEN for Raspberry Pi 3 ではWebアプリからI2C通信に対応したモジュールの制御に[Web I2C API](https://rawgit.com/browserobo/WebI2C/master/index.html) を利用することができる

# 1.準備

## 複数のI2Cモジュールを接続するために

前回はRaspberry Piと温度センサーを4本のジャンパケーブルで直接接続しました。

I2Cバスには複数のモジュールが接続できますので、今回は複数のI2Cモジュールを容易に追加・削除できるように[Grove I2C Hub](http://wiki.seeed.cc/Grove-I2C_Hub/)を利用することにします。

Grove I2C Hubは、4つのGroveコネクタを備えたI2Cモジュールを接続するためのハブです。

4ピンの [Grove 4ピン ケーブル](https://www.seeedstudio.com/Grove-Universal-4-Pin-Buckled-20cm-Cable-5-PCs-pack-p-936.html)を経由すれば、後述する[Grove Digital Light Sensor](http://wiki.seeed.cc/Grove-Digital_Light_Sensor/)などGroveコネクタを備えたI2Cモジュールを直接接続することができます。

Raspberry Pi 3 や前回のADT7410などピンヘッダを備えた（あるいは事前にスルーホールにピンヘッダをはんだ付けしてある）モジュールとの接続には、[Grove 4ピン ジャンパ　メスケーブル](https://www.seeedstudio.com/grove-to-4-pin-254-female-jumper-wire5-pcs-pack-p-1020.html) 経由で接続することができます。

## 用意するもの

ここでは、1つのGroveコネクタつきI2Cモジュールと1つのピンヘッダつきI2Cモジュールを接続することを想定し、下記を用意しておきましょう。

* [CHIRIMEN for Raspberry Pi 3 Hello World](section0.md) に記載の「基本ハードウエア

![ハブとケーブル](imgs/section3/h.png)

* [Grove I2C Hub](http://wiki.seeed.cc/Grove-I2C_Hub/) x 1
* [Grove 4ピン ジャンパー メス　ケーブル](https://www.seeedstudio.com/grove-to-4-pin-254-female-jumper-wire5-pcs-pack-p-1020.html) x 2
* [Grove 4ピン ケーブル](https://www.seeedstudio.com/Grove-Universal-4-Pin-20cm-Unbuckled-Cable-%285-PCs-Pack%29-p-749.html) x 1

上記に加え今回紹介するセンサーが必要となりますが、センサーについては各センサーの説明のパートに記載します。

### 1/10 追加

測距センサーをSRF02からGP2Y0E03しました。

この変更に伴い、下記が追加で必要になります。

* ブレッドボード x 1
* ジャンパーケーブル(オス-オス) x 2

# 2.光センサーを使ってみる

光の強度に反応するセンサーを使ってみましょう。

## a. 部品と配線について
「1.準備」のパートに記載したものに加え、下記を用意してください。

* [光センサー(Grove Digital Light Sensor)](http://wiki.seeed.cc/Grove-Digital_Light_Sensor/) x 1

Raspberry Pi 3との接続方法については、下記回路図を参照ください。

`/home/pi/Desktop/gc/i2c/i2c-grove-light/schematic.png`

![回路図](imgs/section3/k.png)

このセンサーモジュールはGroveコネクタを備えていますので、接続方法に応じてコネクタを選んでください。

* Grove I2C Hub経由で接続する場合 ：Grove 4ピン ケーブル経由で接続してください。
* Raspberry Pi 3へ直接接続する場合：Grove 4ピン ジャンパー メス　ケーブル経由で接続してください。

## b. 接続確認とexampleの実行

i2cdetectで接続を確認しておきましょう。

`$ i2cdetect -y -r 1`

SlaveAddress `0x29` が見つかれば接続OKです。

次にexampleを動かします。

`/home/pi/Desktop/gc/i2c/i2c-grove-light/index.html`

画面の回路図の下の数値が明るさの値です。

センサーに当たる光を遮断してみてください。数値が小さくなるはずです。

逆にセンサーにLEDの光を直接当てると数値が大きくなることが確認できるでしょう。

## c.コード解説

exampleのコードから、光センサーに関係する部分を見ていきます。

今回はドライバーライブラリの中までは深入りせずに、アプリケーションの流れを追ってみましょう。

ADT7410の時とほとんど同じであることがわかるはずです。

### c-1. index.html

下記がindex.htmlの中から主要な部分を抜き出したコードです。

index.html
```html
: 
  <script src="../../polyfill/polyfill.js"></script>
  <script src="../../drivers/i2c-grove-light.js"></script>
  <script src="./main.js"></script>
  :
  <body>
    :
    <p id="head">TEST</p>
    :
  </body>
```

HTMLはADT7410の時とほとんど同じです。
ドライバーライブラリが、`i2c-grove-light.js`に変わりました。

### c-2. main.js

次に、main.jsを見てみましょう。(重要な部分以外は削っています)

main.js
```javascript
navigator.requestI2CAccess().then((i2cAccess)=>{
  var port = i2cAccess.ports.get(1);
  var grovelight = new GROVELIGHT(port,0x29);
  grovelight.init().then(()=>{
    setInterval(()=>{
      grovelight.read().then((value)=>{
        head.innerHTML = value ? value : head.innerHTML;
      });
    },200);
  })
})
```

`main.js`も温度センサーとほとんど同じです。

### var grovelight = new GROVELIGHT(port,0x29)

ここで光センサー用のドライバーライブラリのインスタンス生成を行なっています。

ライブラリ名が変わっただけでADT7410と同様に、`port`オブジェクトと、SlaveAddressをパラメータで渡しています。

### grovelight.init()

`init()`では、インスタンス生成時に指定したportオブジェクトと`slaveAddress(0x29)`を用いて`I2CPort.open()`を行ない、返却される `I2CSlaveDevice` を保存後に`resolve()`で呼び出し元に処理を返しています。

### grovelight.read()

Grove Digital Light Sensorの仕様に基づくデータ読み出し処理をここで実施しています。

# 3.測距センサーを使ってみる (11/10変更)

モノまでの距離を測定する測距センサーを使ってみましょう。

## a. 部品と配線について

「1.準備」のパートに記載したものに加え、下記を用意してください。

* [測距センサー(GP2Y0E03)](http://akizukidenshi.com/catalog/g/gI-07547/) x 1

Raspberry Pi 3との接続方法については、下記回路図を参照ください。

`/home/pi/Desktop/gc/i2c/i2c-GP2Y0E03/schematic.png`

![回路図2](imgs/section3/k2.png)

このセンサーモジュールには、細い7本のケーブルが付属していますが、このままではRaspberry Piと接続することができません。

この細いケーブルを2.54mm のジャンパーピンにハンダづけするなどしてブレッドボード経由でRaspberry Piと接続できるよう、加工しておいてください。

ピンの加工例

![加工例](imgs/section3/k3.jpg)

## b. 接続確認とexampleの実行

i2cdetectで接続を確認しておきましょう。

`$ i2cdetect -y -r 1`

SlaveAddress `0x40` が見つかれば接続OKです。

次にexampleを動かします。

`/home/pi/Desktop/gc/i2c/i2c-GP2Y0E03/index.html`

画面の回路図の下の数値が距離の値(cm)です。
センサーの前面(小さな目玉のような部品が着いた面)を障害物の方向に向けてみてください。障害物とセンサーの距離に応じて数字が変化するはずです。

> GP2Y0E03 が計測できる距離は 60cm くらいまでです。
> 
> 測定できる範囲を超えている場合、out of range と表示されます。

## c.コード解説

exampleのコードから、測距センサーに関係する部分を見ていきます。

### c-1. index.html
下記が`index.html`の中から主要な部分を抜き出したコードです。

index.html
```html
: 
  <script src="../../polyfill/polyfill.js"></script>
  <script src="../../drivers/i2c-GP2Y0E03.js"></script>
  <script src="./main.js"></script>
  :
  <body>
    :
    <p id="distance">init</p>
    :
  </body>
```

HTMLはADT7410の時とほとんど同じです。
ドライバーライブラリが、`i2c-GP2Y0E03.js`に変わりました。

### c-2. main.js
次に、`main.js`を見てみましょう。(重要な部分以外は削っています)

main.js
```javascript
navigator.requestI2CAccess().then((i2cAccess)=>{
  var port = i2cAccess.ports.get(1);
  var sensor_unit = new GP2Y0E03(port,0x40);
  var valelem = document.getElementById("distance");
  sensor_unit.init().then(()=>{
    setInterval(()=>{
      sensor_unit.read().then((distance)=>{
        if(distance != null){
          valelem.innerHTML = "Distance:"+distance+"cm";
        }else{
          valelem.innerHTML = "out of range";
        }
      }).catch(function(reason) {
        console.log("READ ERROR:" + reason);
      });
    },500);
  });
});
```

`main.js`も温度センサーとほとんど同じです。

### var sensor_unit = new GP2Y0E03(port,0x40)

ドライバーライブラリのインスタンス生成処理です。

### sensor_unit.init()

こちらも、内部で`I2CSlaveDevice`インタフェースを取得する処理で、他のセンサーと同様です。

### sensor_unit.read()

測距センサーGP2Y0E03の仕様に基づくデータ読み出し処理をここで実施しています。

# 4.三軸加速度センサーを使ってみる

傾きなどに反応するセンサーを使ってみましょう。

## a. 部品と配線について

「1.準備」のパートに記載したものに加え、下記を用意してください。

* 三軸加速度センサー([GROVE - I2C 三軸加速度センサ ADXL345搭載](http://www.seeedstudio.com/depot/grove-3-axis-digital-accelerometer-adxl345-p-1156.html)) x 1

Raspberry Pi 3との接続方法については、下記回路図を参照ください。

`/home/pi/Desktop/gc/i2c/i2c-grove-accelerometer/schematic.png`

![回路図](imgs/section3/k3.png)

このセンサーモジュールはGroveコネクタを備えていますので、接続方法に応じてコネクタを選んでください。

* Grove I2C Hub経由で接続する場合 ：Grove 4ピン ケーブル経由で接続してください。
* Raspberry Pi 3へ直接接続する場合：Grove 4ピン ジャンパー メス　ケーブル経由で接続してください。

## b. 接続確認とexampleの実行

i2cdetectで接続を確認しておきましょう。

`$ i2cdetect -y -r 1`

SlaveAddress `0x53` が見つかれば接続OKです。

次にexampleを動かします。

`/home/pi/Desktop/gc/i2c/i2c-grove-accelerometer/index.html`

画面の回路図の下に表示されている3つの数値が加速度センサーの値です。

画面左から、X、Y、Zの値となっています。

![加速度センサーの値](imgs/section3/k4.png)

センサーを傾けると数値が変化するはずです。

## c.コード解説

exampleのコードを見てみましょう。

### c-1. index.html

下記が`index.html`の中から主要な部分を抜き出したコードです。

index.html
```html
  : 
  <script src="../../polyfill/polyfill.js"></script>
  <script src="../../drivers/i2c-grove-accelerometer.js"></script>
  <script src="./main.js"></script>
  :
  <body>
    :
      <div id="ax" class="inner">ax</div>
      <div id="ay" class="inner">ay</div>
      <div id="az" class="inner">az</div>
    :
  </body>
```

ドライバーライブラリが、`i2c-grove-accelerometer.js`に、そしてX、Y、Z、3つの値を表示するため要素が3つに変わりましたが、それ以外は今回もこれまでとほとんど同じです。

### c-2. main.js

次に、`main.js`を見てみましょう。(重要な部分以外は削っています)

main.js
```javascript
navigator.requestI2CAccess().then((i2cAccess)=>{
  var port = i2cAccess.ports.get(1);
  var groveaccelerometer = new GROVEACCELEROMETER(port,0x53);
  groveaccelerometer.init().then(()=>{
    setInterval(()=>{
      groveaccelerometer.read().then((values)=>{
        ax.innerHTML = values.x ? values.x : ax.innerHTML;
        ay.innerHTML = values.y ? values.y : ay.innerHTML;
        az.innerHTML = values.z ? values.z : az.innerHTML;
      });
    },1000);
  })
})
```
main.jsも温度センサーとほとんど同じです。

### var groveaccelerometer = new GROVEACCELEROMETER(port,0x53)

ここで加速度センサー用のドライバーライブラリのインスタンス生成を行なっています。

### grovelight.init()

これまでのドライバーライブラリ同様に`init()`では、インスタンス生成時に指定した`port`オブジェクトと`slaveAddress(0x29)`を用いて`I2CPort.open()`を行ない、返却される `I2CSlaveDevice` を保存後に`resolve()`で呼び出し元に処理を返しています。

### groveaccelerometer.read()

`read()` では、加速度センサーの X、Y、Zの値が一度に返却されます。

# 5.演習：複数のセンサーを組み合わせて使ってみよう

せっかくGrove I2C Hubを用意しましたので、これまでの復習と応用を兼ねて下記のような組み合わせで2つのセンサーを繋いで動かしてみましょう。

* 温度センサー(ADT7410)か、「超音波センサー(SRF02)」のどちらか 1つ
* 「光センサー(Grove Digital Light Sensor)」か「三軸加速度センサー」のどちらか１つ

※この組み合わせなら、冒頭で用意したケーブルで足りるはずです。

オンライン版のドライバーライブラリは下記にあります。

* 温度センサー(ADT7410): https://mz4u.net/libs/gc2/i2c-ADT7410.js
* 超音波センサー(SRF02): https://mz4u.net/libs/gc2/i2c-SRF02.js
* 光センサー(Grove Digital Light Sensor): https://mz4u.net/libs/gc2/i2c-grove-light.js
* 三軸加速度センサー : https://mz4u.net/libs/gc2/i2c-grove-accelerometer.js

まずはセンサーを繋いでから、[jsbin](https://jsbin.com/)か[jsfiddle](https://jsfiddle.net/) を使ってコードを書いてみましょう。

# 6.他のI2Cモジュールも使ってみる

前回からこれまでに4つのI2Cセンサーを使ってみました。

本稿執筆中の CHIRIMEN for Raspberry Pi 3 には、他にも `/home/pi/Desktop/gc/i2c/` 配下に下記のようなI2Cモジュールの examples が含まれています。

* i2c-grove-gesture : 「[Grove Gesture](http://wiki.seeed.cc/Grove-Gesture_v1.0/)」(簡単なジェスチャーを判定するセンサー)の接続例です。
* i2c-grove-oledDisplay : 「[Grove OLED Display](https://www.seeedstudio.com/Grove-OLED-Display-0.96%26quot%3B-p-781.html)」(Grove端子で接続できるOLED Display)の接続例です。
* i2c-grove-touch : 「[Grove Touch Sensor](http://wiki.seeed.cc/Grove-I2C_Touch_Sensor/)」(Grove端子で接続できるタッチセンサー)の接続例です。
* i2c-PCA9685 : 「[PCA9685 16-CHANNEL 12-BIT PWM/SERVO DRIVER](https://www.adafruit.com/product/815)」(I2C経由でLEDやサーボモータを16個まで制御可能なモジュール)の接続例です。
* i2c-S11059 : 「[S11059 カラーセンサー](http://akizukidenshi.com/catalog/g/gK-08316/)」(カラーセンサー)の接続例です。
* i2c-VEML6070 : 「[VEML6070 紫外線センサー](https://learn.adafruit.com/adafruit-veml6070-uv-light-sensor-breakout/overview)」(紫外線センサー)の接続例です。
* i2c-multi-sensors : 2つのセンサー（ADT7410とSRF02）を利用する例です。
* i2c-canzasi-blink : 「[Canzasi](https://github.com/tadfmac/Canzasi)」(筆者が開発を進めるI2C Slave Device開発環境) の接続例です。

ご興味がありましたら、ぜひ触ってみてください。

# まとめ

このチュートリアルでは 下記について学びました。

* Grove I2C Hubを使ったI2Cモジュールの接続方法
* 2.光センサーの使い方
* 3.超音波センサーの使い方
* 4.3軸加速度センサーの使い方
* 複数のセンサーを繋いだ演習

次回のCHIRIMEN for Raspberry Pi 3 チュートリアルはいよいよ最終回！

Web GPIO APIとWeb I2C APIを組み合わせたプログラミングに挑戦してみたいと思います。
お楽しみに！