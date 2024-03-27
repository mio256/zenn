---
title: "wlan.connect が 繋がらない"
emoji: "😡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "micropython", "wlan", "raspi"]
published: true
---

## 状況

Raspberry Pi Pico W でインターネットに接続する際、`wlan.connect` が繋がらない。

公式 [Raspberry Pi Pico W でインターネットに接続する](https://datasheets.raspberrypi.com/picow/connecting-to-the-internet-with-pico-w.pdf) のサンプルコードをそのまま使うと、`wlan.isconnected()` がいつまで経っても `False` になる。

```py
wlan.connect('Wireless Network', 'The Password')
```

## 解決策

PDFのサンプルコードをそのままコピーすると、`'`（0x27） が `’`（0x60） になってしまうので、`"` に直す。

```py
wlan.connect("Wireless Network", "The Password")
```

## 一言

こんなことに3時間も苦しんだ…

PDFのコピペは注意が必要。

## 参考

https://tamasan238.hatenablog.com/entry/2023/07/20/233351

https://www.fourier.jp/techblog/articles/play-with-raspbery-pi-pico-w/

https://datasheets.raspberrypi.com/
