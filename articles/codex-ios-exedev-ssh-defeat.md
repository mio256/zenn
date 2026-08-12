---
title: "【敗北】Codex iOSからexe.devへSSHできない"
emoji: "🏳️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["codex", "ios", "ssh", "exedev"]
published: true
---

## TL;DR

- Codex iOS Appからexe.devのVMへSSH接続すると、約500msでDisconnectする
- 同じiPhone・ホスト・ユーザー・秘密鍵を使ったTermiusからは接続できる
- Ed25519とECDSA P-256の秘密鍵を試したが、Codex iOSだけ失敗した
- Retry中にVMを監視しても、新しいSSHセッションは生成されなかった
- Codex CLIやapp-serverへ到達する前、SSH認証より前の段階で切断されている
- exe.devが提示するRSAホスト証明書とCodex iOSの互換性を疑っているが、原因は確定できなかった。敗北

## そもそもexe.devとは

[exe.dev](https://exe.dev/)は、SSHを操作インターフェースにしたVMホスティングサービスです。

```bash
# VMを作る
ssh exe.dev new

# VM一覧を見る
ssh exe.dev ls

# VMへ入る
ssh <vm-name>.exe.xyz
```

この体験が本当に良いです。

数秒で永続Ubuntu VMが生え、SSHで普通のLinuxとして触れます。アプリを起動すればHTTPS付きのURLで公開でき、認証やドメイン周りもかなり面倒を見てくれます。それでいて、独自のデプロイDSLに閉じ込められるのではなく、最終的に触っているのは普通のVMです。

AIコーディングエージェントとの相性も抜群です。「アイデアごとに計算機を1台生やし、エージェントを放り込み、そのまま公開する」という雑で強い運用ができます。個人開発でありがちな、コードはできたのにデプロイが面倒でローカルに眠る問題をかなり減らしてくれます。

今回2時間以上デバッグしてなお、exe.devをやめようとは一度も思いませんでした。それくらい気に入っています。本記事はexe.devへの不満ではなく、Codex iOSからもつながったら最高なのに、という話です。

## 直面した現象

Codex iOS AppのSSH Hostに以下を設定しました。

- Host: `<vm-name>.exe.xyz`
- Port: `22`
- Username: exe.devのユーザー名
- Authentication: Private key

接続すると、約500msで次のエラーになります。

```text
Could not connect to <user>@<vm-name>.exe.xyz over SSH.
```

詳細ログは表示されません。

一方、同じiPhone上のTermiusへ、同じHost / Port / Username / Private keyを設定すると普通に接続できました。

```bash
echo hello
```

も成功します。

つまり、少なくとも次は正常です。

- iPhoneからVMまでのネットワーク
- 秘密鍵
- exe.devへの公開鍵登録
- exe.dev側のSSHサービス

問題はCodex iOS固有に見えます。

## 最初に疑ったもの

### Codex CLIのバージョン差

VMにはCodex CLIが2つありました。

```bash
ssh <user>@<vm-name>.exe.xyz \
  'command -v codex; codex --version; ~/.local/bin/codex --version'
```

```text
/usr/local/bin/codex
codex-cli 0.144.5
codex-cli 0.147.0
```

非対話SSHでは古い`0.144.5`が見えていました。

Codex iOSはSSH接続後、リモートの`codex`を探してapp-serverへ接続します。そのため、最初はCLIとiOS Appのプロトコル不一致を疑いました。

しかし、実際に動いていたapp-serverを確認すると`0.147.0`でした。

```bash
ssh <user>@<vm-name>.exe.xyz \
  'codex app-server daemon version'
```

```json
{
  "status": "running",
  "cliVersion": "0.144.5",
  "appServerVersion": "0.147.0"
}
```

### 古いUnixソケット

次に、古いソケットやロックファイルが残っている可能性を疑いました。

```bash
ssh <user>@<vm-name>.exe.xyz \
  'ls -la ~/.codex/app-server-control; ss -xlpn | grep app-server'
```

しかしapp-serverプロセスは生存しており、ソケットも`LISTEN`状態でした。

さらに、古いCLIと新しいCLIの両方からWebSocketハンドシェイクを試しました。

```bash
{
  printf 'GET / HTTP/1.1\r\n'
  printf 'Host: localhost\r\n'
  printf 'Upgrade: websocket\r\n'
  printf 'Connection: Upgrade\r\n'
  printf 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==\r\n'
  printf 'Sec-WebSocket-Version: 13\r\n\r\n'
  sleep 1
} | timeout 4 /usr/local/bin/codex app-server proxy
```

結果は正常でした。

```text
HTTP/1.1 101 Switching Protocols
connection: Upgrade
upgrade: websocket
```

`~/.local/bin/codex app-server proxy`でも同じ結果です。

app-server、Unixソケット、CLIのバージョン差は今回の直接原因ではなさそうです。

### netcatの`-U`対応

Codex iOSのLinux向けSSH接続では、Unixソケットへの接続に`nc -U`が使われることがあります。Debianの`nc.traditional`には`-U`がなく、Codex iOSだけ接続できない既知事例があります。

```bash
nc -h 2>&1 | head
```

今回のVMにはOpenBSD netcatが入っており、`-U`も利用可能でした。

そもそも後述の監視結果では、今回のRetryは`nc`を起動する段階まで到達していません。

## Retry中のプロセスを50ms間隔で監視した

ここが一番大きな切り分けでした。

Codex iOSでRetryを押すタイミングに合わせて、VM上で`sshd-session`と`codex app-server proxy`の新規プロセスを50ms間隔で監視しました。

```bash
end=$((SECONDS + 60))

while test "$SECONDS" -lt "$end"; do
  ps -eo pid=,ppid=,comm=,args= |
    awk '$3 == "codex" && $0 ~ /app-server proxy/ { print }
         $3 == "sshd-session" && $0 ~ /<remote-user>/ { print }'
  sleep 0.05
done
```

RetryするとiOS側は約500msでDisconnectしましたが、VMには新しい`sshd-session`も`codex app-server proxy`も現れませんでした。

対照実験としてOpenSSHやTermiusから接続すると、`sshd-session`はすぐに生成されます。

したがって、今回の失敗は次より前です。

```text
SSHの鍵交換・ホスト鍵検証
  ↓
ユーザー認証
  ↓
sshd-session生成
  ↓
Codex bootstrap
  ↓
app-server proxy
```

少なくとも、Codex CLI、PATH、app-server、Unixソケット、`nc -U`を直しても、今回のRetry経路には届きません。

## ユーザー鍵をEd25519からECDSAへ変えた

Codex iOSがEd25519のPKCS#8秘密鍵を正しく扱えていない可能性も疑いました。

最初に使ったのは、暗号化なしのEd25519 PKCS#8鍵です。

```bash
openssl genpkey -algorithm ed25519 -out key.pem
```

そこで一変数だけ変え、暗号化なしのECDSA P-256 SEC1鍵を作りました。

```bash
openssl ecparam -name prime256v1 -genkey -noout \
  -out codex-ios-ecdsa-p256-test.pem

ssh-keygen -y \
  -f codex-ios-ecdsa-p256-test.pem \
  > codex-ios-ecdsa-p256-test.pub
```

exe.devへの公開鍵登録はSSH APIだけで完結します。

```bash
cat codex-ios-ecdsa-p256-test.pub |
  ssh exe.dev ssh-key add
```

登録したECDSA鍵だけを使い、macOS OpenSSHから接続できることも確認しました。

```bash
ssh \
  -o BatchMode=yes \
  -o IdentitiesOnly=yes \
  -o IdentityAgent=none \
  -i codex-ios-ecdsa-p256-test.pem \
  <user>@<vm-name>.exe.xyz \
  'echo ECDSA_TEST_OK'
```

```text
ECDSA_TEST_OK
```

しかし、この秘密鍵をCodex iOSへ設定しても、同じく約500msでDisconnectしました。

Ed25519固有の問題でもなさそうです。

## exe.devが提示するホスト鍵

OpenSSHのデバッグログを確認しました。

```bash
ssh -vvv <user>@<vm-name>.exe.xyz true
```

要点は以下です。

```text
Remote protocol version 2.0, remote software version Go
kex: algorithm: ecdh-sha2-nistp256
kex: host key algorithm: rsa-sha2-512-cert-v01@openssh.com
Server host certificate: ssh-rsa-cert-v01@openssh.com
Server host certificate ID "exe-dev-host"
```

ホスト鍵のフィンガープリントは次です。

```text
SHA256:JJOP/lwiBGOMilfONPWZCXUrfK154cnJFXcqlsi6lPo
```

これは[exe.dev公式ドキュメントに掲載されているフィンガープリント](https://exe.dev/docs/faq/host-key)と一致します。怪しい中間者がいるわけではなく、exe.dev標準のSSH入口です。

サーバーは次のホスト鍵アルゴリズムを提示していました。

```text
rsa-sha2-256
rsa-sha2-512
ssh-rsa
rsa-sha2-256-cert-v01@openssh.com
rsa-sha2-512-cert-v01@openssh.com
ssh-rsa-cert-v01@openssh.com
```

OpenSSHは`rsa-sha2-512-cert-v01@openssh.com`を選択し、正常に接続します。

## 現時点の推測

Codex iOSのSSH実装と、exe.devのRSA OpenSSHホスト証明書の間に互換性問題があるのでは、と疑っています。

例えば、Codex iOS側が証明書形式のホスト鍵アルゴリズムを選択したあと、証明書の解析や検証で失敗している可能性です。

ただし、iOS側の詳細ログを取得できていないため、これは推測です。Codex iOSには`HostKeyAlgorithms`を指定するUIもないため、exe.devが提示する通常のRSAホスト鍵へ切り替える比較テストもできませんでした。

確定できたのは、以下までです。

- 同じiPhoneと秘密鍵でTermiusは成功する
- Ed25519でもECDSAでもCodex iOSは失敗する
- iOSのRetryはVM上にSSHセッションを生成しない
- Codex bootstrapやapp-serverより前で失敗している
- exe.devはRSA OpenSSHホスト証明書を提示している

原因を特定して直すところまでは届きませんでした。敗北です。

## References

- [exe.dev](https://exe.dev/)
- [exe.dev Docs](https://exe.dev/docs/all)
- [What is the host key for exe.dev?](https://exe.dev/docs/faq/host-key)
- [exe.dev の使い方と、導入メリット](https://zenn.dev/katsuhisa_/articles/exe-dev-guide)
- [exe.devが切り拓くオンデマンドソフトウェア時代の可能性](https://zenn.dev/ktnyt/articles/df7ed0df1e65b3)
- [ハーネスエンジニアリングに挑戦し、AIテニスコーチアプリをAIと作った話](https://zenn.dev/zerebom/articles/ba3e80a21eef63)

最後に、調査内容は[openai/codex#38264](https://github.com/openai/codex/issues/38264)へ報告しました。

SSH、iOS、OpenSSHホスト証明書あたりに詳しい有識者の方がいたら、助けてください。
