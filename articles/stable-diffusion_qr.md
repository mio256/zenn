---
title: "StableDiffusionでQRコードをアートにできるってマ！？"
emoji: "📍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["StableDiffusion", "AI", "Python", "機械学習", "画像生成"]
published: true
---

## はじめに

どうやら、StableDiffusionでQRコードを装飾し、アートにできるらしい。

https://twitter.com/makotofalcon/status/1665959493673553921

![](https://storage.googleapis.com/zenn-user-upload/a3adeca16b4c-20230606.png)
![](https://storage.googleapis.com/zenn-user-upload/ac01288d6e0a-20230606.jpeg)

## 調べてみる

バズっているのはこちらのReddit

https://www.reddit.com/r/StableDiffusion/comments/141hg9x/controlnet_for_qr_code/

元ネタはおそらくこちらの論文

https://mp.weixin.qq.com/s/i4WR5ULH1ZZYl8Watf3EPw

どうやら「ControlNet for QR Code」という名称のControlNetのモデル？みたいだ。

現在はHuggingFaceのリポジトリが公開停止されているため、OSSとして公開されるのかどうかはわからない。

## 再現しよう！

こちらの投稿によると、アート性は欠けるがControlNetの標準機能でそれっぽいものは作れるようだ。

https://www.reddit.com/r/StableDiffusion/comments/141hg9x/comment/jn3l84q/?utm_source=share&utm_medium=web2x&context=3

StableDiffuison WebUI Automatic1111 + Control-Net が導入されていることを前提とするため、未導入の方は超絶わかりやすい「てるろび」さんの動画を参考に導入してね💕

https://youtu.be/8QIJMGW_-LM

https://youtu.be/5uGt_t4t1iU

### プロンプト

モデル、プロンプト、サンプリングメソッドは適当に指定する。

高速でチャチャッと生成したければサンプリングメソッドはDDIMがおすすめ。

私の環境は以下のようにした。

```
Colorful
Steps: 15, Sampler: DDIM, CFG scale: 7, Seed: 711902182, Size: 512x512, Model hash: 6ce0161689, Model: v1-5-pruned-emaonly, Version: v1.3.0
```

![](https://storage.googleapis.com/zenn-user-upload/a10c24db8704-20230607.png)


### ControlNet

ここからが本題である。

QRコードを用意し、画像のように設定する。

```
Enable

Preprocesser    inpaint_global_hamonious
Model           control_*_*_inpaint

Control Weight      0.65
Ending Control Step 0.85
```

![](https://storage.googleapis.com/zenn-user-upload/7b9e726ccf59-20230607.png)

### 動作確認

出来上がったQRコードがこちらである。

![](https://storage.googleapis.com/zenn-user-upload/1b476c26624a-20230607.png)

しっかりとQRコードを読み取ることができる。

比較用に下に本家のQRコードを載せておく。

![](https://storage.googleapis.com/zenn-user-upload/4a14a74087ea-20230606.jpeg)

### アート性どこやねん

おっしゃるとおり、本家のようなアート性はなく「QRコードの黒い部分に絵を書いてみました」になってしまっている。

なぜならば、ControlNetで使用した「inpaint」は黒く指定された部分を再度生成するというものであり、本来は気に入らない部分だけを指定してやり直す用の機能である。

それをQRコードに適応させているため、もちろん黒い部分しか生成されていない。

プロンプトやモデルの工夫で多少はアート性を出すことはできそうだが、本家のような圧巻のアートを生み出すことは難しそうだ。

## 兄貴、OSS待ってます！

まだ論文が公開されてから数日しか経っていないため ControlNet for QR Code がこれからどうなるかはわからない。

作者からの朗報を待とう！

https://github.com/ciaochaos

よくわからない部分があれば気軽にコメントを！

また、本記事に間違いなどがあったらぜひコメントやIssueをお願いします🙏
