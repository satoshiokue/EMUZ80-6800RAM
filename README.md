# EMUZ80-6800RAM

![EMUZ80-6809RAM](https://github.com/satoshiokue/EMUZ80-6800RAM/blob/main/MEZ6800RAM.jpeg)  
6800 Single-Board Computer    

電脳伝説さん(@vintagechips)のEMUZ80が出力するZ80 CPU制御信号をメザニンボードで組み替え、6800と64kB RAMを動作させることができます。  
RAMの制御信号とメモリマップドIOのMRDY信号をPICのCLC(Configurable Logic Cell)機能で作成しています。  
電源が入るとPICは/HALT信号で6800を停止させ、データをRAMに転送します。データ転送完了後、バスの所有権を6800に渡してリセットを解除します。  

MC6800LとPIC18F47Q83の組み合わせで動作確認しています。  

動作確認で使用したMPU  
MC6800L (1MHz - 2MHz)  
EF6800P  
MC68A00P  
HD46800P  

このソースコードは電脳伝説さんのmain.cを元に改変してGPLライセンスに基づいて公開するものです。

## メザニンボード
https://github.com/satoshiokue/MEZ6800RAM


## 回路図
https://github.com/satoshiokue/MEZ6800RAM/blob/main/MEZ6800RAM.pdf

## ファームウェア

EMUZ80で配布されているフォルダemuz80.X下のmain.cと置き換えて使用してください。
* emuz80_6800ram.c  

## クロック周波数

172行目のCLK_6800がクロック周波数です。初期値は2MHzになっています。  
```
#define CLK_6800 2000000UL	// 6800 clock frequency(Max 16MHz) 2MHz=2000000UL
```
NCO機能でクロック源を生成して、CWG機能で2相クロックを生成します。  
6800がIO空間をアクセスするとCLCでCWGからNCOのクロック源を切り離します。これによってE信号がHighの状態を延長することでWAITとします。  

## アドレスマップ
```
Memory
RAM1  0x0000 - 0x7FFF 32kバイト
RAM2  0x9000 - 0xFFFF 28kバイト

I/O   0x8000 - 0x8FFF
制御レジスタ 0x8018
通信レジスタ 0x8019


BASICワークエリア 0x0000 - 0x7FFF
モニターワークエリア 0x9000 - 0xBFFF
```

## PICプログラムの書き込み
EMUZ80技術資料8ページにしたがってPICに適合するファイルを書き込んでください。  

またはArduino UNOを用いてPICを書き込みます。  
https://github.com/satoshiokue/Arduino-PIC-Programmer

PIC18F47Q43 emu6800RAM_1MHz_Q43.hex  

PIC18F47Q83 emu6800RAM_1MHz_Q8x.hex  
PIC18F47Q84 emu6800RAM_1MHz_Q8x.hex  

## メモリーイメージについて
PICファームウェアは以下のアドレスにデータを転送してUniversal Monitorが起動します。
```
0x0000 ALTAIR 680 BASIC   Start 0x0000
0xE000 Mikbug             Start 0xE0D0
0xE800 Mikbug 2.0         Start 0xE800
0xF000 Universal Monitor  Start 0xF000
```

それぞれのモニターから0x0000にジャンプするとBASICが起動します。  
メモリーサイズはENTERキーを押すことで確保可能な最大メモリーを確認して設定されます。  
```
Universal Monitor 6800
MC6800
] G0000

MEMORY SIZE?
TERMINAL WIDTH? 80
WANT SIN-COS-TAN-ATN? Y

26330 BYTES FREE

MITS ALTAIR 680 BASIC VERSION 1.1 REV 3.2
COPYRIGHT 1976 BY MITS INC.

OK
```

Mikbugの変数は0xA000から0xE200へ移動しました。Gコマンドを使用する際は0xE248,0xE249へアドレスを設定してください。  
Mikbug 2.0のワークエリアはオリジナルの0xA000-です。  

Universal Monitor  
https://electrelic.com/electrelic/node/1317

## 6809プログラムの格納
インテルHEXデータを配列データ化して配列rom[]に格納すると0xE000に転送され6800で実行できます。

## 参考）EMUZ80
EUMZ80はZ80CPUとPIC18F47Q43のDIP40ピンIC2つで構成されるシンプルなコンピュータです。

![EMUZ80](https://github.com/satoshiokue/EMUZ80-6502/blob/main/imgs/IMG_Z80.jpeg)

電脳伝説 - EMUZ80が完成  
https://vintagechips.wordpress.com/2022/03/05/emuz80_reference  
EMUZ80専用プリント基板 - オレンジピコショップ  
https://store.shopping.yahoo.co.jp/orangepicoshop/pico-a-051.html

## 参考）phemu6809
comonekoさん(@comoneko)さんがEMUZ80にMC6809を搭載できるようにする変換基板とファームウェアphemu6809を発表されました。

![phemu6809](https://github.com/satoshiokue/EMUZ80-6502/blob/main/imgs/IMG_6809.jpeg)

https://github.com/comoneko-nyaa/phemu6809conversionPCB  
phemu6809専用プリント基板 - オレンジピコショップ  
https://store.shopping.yahoo.co.jp/orangepicoshop/pico-a-056.html
