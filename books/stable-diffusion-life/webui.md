---
title: "WebUI AUTOMATIC1111の導入"
---

僕たちは機械語を話せないので、大人しくUIを使いましょう。

https://github.com/AUTOMATIC1111/stable-diffusion-webui

こちらのAUTOMATIC1111さんのStableDiffuison WebUIを使わせていただきます。

コマンドラインなどに慣れていない方は、てるろびさんの動画が一つ一つの手順を丁寧に解説されているのでおすすめです！

https://youtu.be/8QIJMGW_-LM

## インストール

[Automatic Installation on Windows](https://github.com/AUTOMATIC1111/stable-diffusion-webui#automatic-installation-on-windows)を参考にインストールを進めていきましょう！

2023/06/06時点では以下のような手順です。

1. `Python3.10.6`をダウンロードし、`Add Python to PATH`にチェックを入れてインストール
2. `git`をインストール
3. StableDiffuison WebUIをダウンロードしたいディレクトリで`git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git`を実行

## 実行

`webui-user.bat`を実行することで、WebUIを起動することができます。

```sh:powershell
> cd stable-diffusion-webui
> ./webui-user.bat
```

## 操作

基本操作については、こちらの記事が詳しくまとまっているのでおすすめです。

https://freeblog-video.com/stable-diffusion_basic/

<!-- TODO 操作画面画像 -->

## テスト実行

とりあえず、それっぽいものを生成してみましょう。

猫が寝ている画像を生成してみます。

ポジティブプロンプトには生成してほしいものを書き、ネガティブプロンプトには生成してほしくないものを書きます。

```:PositivePrompt
```

```:NegativePrompt
```

<!-- TODO 操作画面 -->

<!-- TODO 実行結果 -->

寝ている猫の画像が生成できましたね！

## エラー集

実行時にこのようなエラーが出た場合は、`webui-user.bat`のオプションに`--xtransformer`を追加すると治る場合があります。

```:error

```

```:webui-user.bat

```

<!-- TODO エラー分析; エラー内容、編集内容を追記 -->
