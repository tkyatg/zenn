---
title: "Gopher くん NFT ガチャガチャサイト作ってみた"
type: "tech"
topics: ["個人開発","ポエム","nextjs","solidity","vercel"]
emoji: "💊"
published: false
---

こんにちは、暇があれば個人開発している厚木です。
今回は Solidity & Next.js を勉強がてら Free Mintサイトを作った話です。
※テストネットなので、誰でも無料で試せます。

## 作ったもの

↓早速どんなものを作ったかのイメージ

<https://free-mint-gopher-front-git-main-tkyatg.vercel.app/>
![image](/images/articles/tech/free-mint-gopher/init.png)

誰でも Mint できて、4種のランダムな Gopher くん NFT が出現します。
ちなみにサイトで使っている Gopher くんは、すべて私が作りました（10分くらいで）

※オリジナルの The Go gopher（Gopher くん）は、Renée French によってデザインされました。

## リポジトリ

今回のソースコードは、以下リポジトリにすべて入っています。
※言い訳ですが、バックエンド本業なので、フロントコード汚いです。申し訳ありません。

[nextjs](https://github.com/tkyatg/free-mint-gopher-front)
[solidity](https://github.com/tkyatg/free-mint-gopher-contract)

## 実際に使ってみる

<https://free-mint-gopher-front-git-main-tkyatg.vercel.app/>

### 初期表示

![image](/images/articles/tech/free-mint-gopher/init.png)

### 認証

Walletごとに認証方法が異なっていたりするので、wallet connect を採用しています。
スマホに入れている Wallet で QRcode を読み込んで、承認すると認証完了です。

![image](/images/articles/tech/free-mint-gopher/auth.png)

※wallet connect は 2023/3月に v2 にしないと使えなくなるようです。

### ガチャを回す

ガチャを回すボタンを押すと、認証した wallet に sign 要求が送信されます。
wallet で承認すると、transaction が開始されます。
![image](/images/articles/tech/free-mint-gopher/sending.png)

Mint には Goerli が必要になります。
↓のマイニングサイトで無料で手に入ります。

<https://goerli-faucet.pk910.de/>

### ガチャガチャで取得した NFT 表示

transaction が完了すると、当たった Gopher くん NFT が表示されます。
混み具合によりますが、大体 10 秒前後で表示されます。

![image](/images/articles/tech/free-mint-gopher/gopher.png)

ちなみに表示されるNFTは完全ランダムです。

## Web3の技術に関して思うこと

- Dapps はすごく面白いが、色々なところで処理が遅すぎる
  - Mint した NFT を表示するまで 10-15 秒くらいかかる
  - 業界的に有名な web3 idaas を使うと認証に10秒かかる
- 外部パッケージなどを使う場合は、セキュリティをめちゃくちゃ気にする必要がある
  - 毎度セキュリティ的にどうかを判断するのがコスト
- とはいえ世界に電子データアートなどを販売する際に NFT にして売るのは超良い
