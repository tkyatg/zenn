---
title: "【Golang】料率計算に Float 型を使ってはならない"
type: "tech"
topics: ["go"]
published: true
emoji: "🐱"
published_at: 2023-02-22 23:10
---

どうも、レベチの Gopher を目指している厚木です。
現在は、Go で金融系のサービス開発に携わっています。

今回は、金融系の開発において必須の「料率計算を Go で実装した話」です。

### 実装概要

今回は、税込価格から税抜価格・消費税を求める実装をしました。

例えば
税込121 円 → 税抜110 円 + 消費税 11 円
となります。

上記の内容を Go で実装してみましょう。

## Float 型を使って税金額を算出してみる

Go では Int で小数点は扱えないので、Float で実装してみます。
税抜価格を算出し、税込価格と税抜価格の差から消費税を求めます。

```go
package main

import (
 "fmt"
 "math"
)

func main() {
 taxRate := float64(10)
 taxIncludedPrice := float64(121)

 // 消費税が切り捨てなので、税抜価格は切り上げで計算する
 // 110円になってほしい
 taxExcludedPrice := math.Ceil(taxIncludedPrice / float64(100+taxRate) * 100)

 // 11円になってほしい
 tax := taxIncludedPrice - taxExcludedPrice

 fmt.Println(tax)
}
```

実装自体に間違いはなさそうに見えますが
実際に `go run main.go` で実行すると、結果は `10` と算出されます。

## なぜ Float 型ではズレるのか

Float(浮動小数点数) では、表現できる値の範囲が決まっており、表現しきれない値は「なるべく近い値」で表現します。
そのため Float を用いた計算では多少のずれが生じます。
※丸め誤差

誤差に関しては、以下の記事が参考になります。
<https://qiita.com/sonatard/items/eac6fb35dcc8e052a293#%E8%AA%A4%E5%B7%AE>

サンプルコードの場合、`taxExcludedPrice` は`110.000001...`のようになります。
切り上げると `111` となり、消費税は `10` となってしまうわけです。

サンプルコードの tax のズレは最大 `1` ですが、100 万回の決済が起これば、100 万円損するような大きなバグです。

少しのズレも許容できない計算を行いたい場合は、浮動小数点数ではなく、小数点数を使う必要があります。

## math/big.Rat で計算する

いくつか方法があるのですが、今回は `math/big.Rat` を使ってズレがないように計算します。
`math/big.Rat` は有理数を扱うためのパッケージです。
<https://pkg.go.dev/math/big>

```go
package main

import (
 "fmt"
 "math/big"
)

func main() {
 taxRate := int64(10)
 taxIncludedPrice := int64(121)

 taxIncludedPriceRat := big.NewRat(taxIncludedPrice, 1)

 taxExcludedPriceRat := new(big.Rat).Quo(taxIncludedPriceRat, big.NewRat(100+taxRate, 100))

 taxExcludedPrice, remainder := new(big.Int).DivMod(taxExcludedPriceRat.Num(), taxExcludedPriceRat.Denom(), new(big.Int))
 if remainder.Cmp(big.NewInt(0)) > 0 {
  taxExcludedPrice.Add(taxExcludedPrice, big.NewInt(1))
 }

 tax := taxIncludedPrice - int64(taxExcludedPrice.Uint64())

 fmt.Println(tax)
}
```

消費税の算出方法ですが、Float の時と同じく税抜価格を算出し、税込価格と税抜価格の差から消費税を求めます。
実行結果は `11` となります。

上記のように有理数を使うことで、内税価格から消費税を算出することができます。

## まとめ

少しのズレも許容できない計算には Float を使うとバグを生みます。
Go の場合は、Intでは小数点が切り捨てられるので、`math/big`などのパッケージを使いましょう。

あと業務委託のお仕事募集してます
連絡先: takuya.atsugi911@gmail.com
