---
title: "Tensorflowしたかっただけなのに、SSL:CERTIFICATE_VERIFY_FAILEDで詰んだ"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","ssl","tensorflow"]
published: true
---

Python3環境で以下のエラーが出て、久々に詰まりまくったので備忘録。

```
Exception: URL fetch failure on https://storage.googleapis.com/tensorflow/tf-keras-datasets/mnist.npz: None -- [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:1007)
```

動作環境
```
PC : M1 MacBook Pro (Apple M1)
python
- python3.12
- python3.11
- python3.10
```

「venvが悪いのか、pyenv入れるぞ」
「securityに指定されているのはpython3.10までだから、python3.10でやれば動くか！？」
と右往左往したが、結局エラーに書いてあるとおりSSL証明がうまくいっていないだけである。

以下のコードを実行し、SSL/TLS証明書の検証を無効にすればよいだけである。

```python
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

今回はMNISTをダウンロードしたかっただけなのでSSL/TLS証明書の検証を無効にしたが、セキュアな接続が必要な場面でこの対処はしないほうが良さそう。

もっとも、DjangoやFlaskにおいてこのエラーが発生することは考えにくいが…


## 参考

https://stackoverflow.com/questions/35569042/ssl-certificate-verify-failed-with-python3
