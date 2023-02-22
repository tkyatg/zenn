---
title: "【Golang】料率計算に Float 型を使ってはならない"
type: "tech"
topics: ["go"]
published: true
emoji: "🐱"
published_at: 2023/01/25 09:00:00
---

どうも、レベチの Gopher を目指している厚木です。
現在は、Go で金融系のサービス開発に携わっています。

金融サービスでは、ままあることですが
商品の内税額から、税抜価格・税額は幾らかを計算します。

例えば、商品価格が内税で 121 円・税率 10% の場合
税抜き価格が 110 円、税金が 11 円になります。

上記の内容を Go で実装してみましょう。

## Float 型で料率計算してみる

まずは Float で実装してみます。
税抜価格を算出し、税込価格と税抜価格の差から税金額を求めます。

```go
package main

import (
 "fmt"
 "math"
)

func main() {
 taxRate := float64(10)
 taxIncludedPrice := float64(121)

 // 税金額が切り捨てなので、税抜価格は切り上げで計算する
 taxExcludedPrice := math.Ceil(taxIncludedPrice / float64(100+taxRate) * 100)

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

上記サンプルコードの場合、`taxExcludedPrice` が `110` となってほしいところですが
浮動小数点数で計算すると`110.000001...`のようになります。

切り上げると `111` となり、税金額は `10` となってしまうわけです。

実際にサンプルコードの tax のズレは最大 `1` ですが
100 万回の決済が起これば、100 万円損する大きなバグです。

少しのズレも許容できない計算を行いたい場合は
浮動小数点数ではなく、小数点数を使う必要があります。

## math/big.Rat　で計算する

今回は `math/big.Rat` を使ってズレがないように計算します。
`math/big.Rat` は有理数を扱うためのパッケージです。

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

Float の時と同じく、税抜価格を算出し、税込価格と税抜価格の差から税金額を求めています。
実行結果は `11` となります。

## まとめ

少しのズレも許容できない計算には Float を使うな。

あと業務委託のお仕事募集してます
連絡先: takuya.atsugi911@gmail.com
