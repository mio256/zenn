---
title: "go cleanしたらストレージが15GB空いた話"
emoji: "😲"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go", "clean", "storage", "mac"]
published: true
---

## はじめに

ケチって256GBのMacbookを買ったら、詰んだ。

![](/images/go-cleanup-cmd/p1.png)

## 環境

```
PC : M1 MacBook Pro (Apple M1)
Storage: 256GB
```

## すでにやったこと

真っ先に行うことといえば、写真や動画をクラウドに保存することだろう。
しかし、私の友達はMacbookくらいしかいないので、インスタ映えしそうな大量の写真や動画などはなかった。

![](/images/go-cleanup-cmd/image.png)

- ゴミ箱を空にする
- XCodeをアンインストール → 約10GB
- ローカルのDockerイメージを削除 → 約5GB
- ローカルのプロジェクトソースを削除 → 約5GB
- brew cleanup → 約2GB

普通であれば、このくらいでなんとかなると思うが、まだまだ足りない。

追加で必要な開発環境をいくつか構築すると、またすぐに詰んだ。

## go clean

以下の2つのコマンドでgoのキャッシュを削除することができる。

```
go clean -cache
go clean -modcache
```

これでかなり助かった。

![](/images/go-cleanup-cmd/p2.png)

## 参考

https://zenn.dev/spiegel/articles/20210223-go-module-aware-mode
