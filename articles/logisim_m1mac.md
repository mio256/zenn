---
title: "Logisim を M1 Mac で動かす"
emoji: "☕️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Java", "Logisim", "Mac", "M1"]
published: true
---

LogisimをM1Macで動かしたいと思ったことはありますか？

ありますよね？

ありますね？

動作環境

```
PC : M1 MacBook Pro (Apple M1)
Java : openjdk 11.0.19 2023-04-18
```

## Logisimのインストール

[公式サイト](https://sourceforge.net/projects/circuit/) の [ファイル一覧](https://sourceforge.net/projects/circuit/files/2.7.x/2.7.1/) から `logisim-generic-2.7.1.jar` をダウンロードしてください

※ 最新バージョンがリリースされていたらそちらを使用することをおすすめします

![](/images/logisim/2023-06-28_21.37.42.png)

## Javaのインストール

homebrewが入っていなければ入れてください

```sh
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

$ brew -v
Homebrew 4.0.22
```

javaが入っていなければhomebrewでjavaを入れてください

```sh
$ brew install java

$ java --version
openjdk 11.0.19 2023-04-18
```

## Logisimの起動

```sh
# ダウンロードしたディレクトリに移動
$ cd ~/Downloads

# ダウンロードファイルがあることを確認
$ ls
logisim-generic-2.7.1.jar ...

# Javaで実行
$ java -jar logisim-generic-2.7.1.jar
```

以下のようなウィンドウが表示されたら成功です

![](/images/logisim/2023-06-28_21.54.47.png)
