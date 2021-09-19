---
title: "【sinon.js】mockとstubの使い分け.md"
date: 2021-09-12T23:24:42+09:00
draft: false
tags: ["node"]
---

こんにちは、[かわそん](https://twitter.com/KKohey4)です。

sinon.jsを使っていて、mockと stub の区別がつかなかったので、個人的基準を共有します。

<!--more-->
## 結論
- 基本、mockを使えばいい
- 条件分岐の中に、意図的に入りたい場合は、stubを使えばいい

## mock・stubとは？
公式ドキュメントより引用します

`mock`
```
Mocks (and mock expectations) are fake methods (like spies) with pre-programmed behavior (like stubs) as well as pre-programmed expectations.

A mock will fail your test if it is not used as expected.
```

訳すと、

```
モックは、事前にプログラムされた動作と、事前にプログラムされた期待値を持つ偽のメソッドです。

モックは、期待通りに使用されないとテストに失敗します。
```
ほむ、、、  
stubは？

```
Test stubs are functions (spies) with pre-programmed behavior.

They support the full test spy API in addition to methods which can be used to alter the stub’s behavior.

As spies, stubs can be either anonymous, or wrap existing functions. When wrapping an existing function with a stub, the original function is not called.
```

```
テストスタブは、あらかじめ動作がプログラムされた関数（スパイ）です。

スタブは、テストスパイの完全な API をサポートしているほか、 スタブの挙動を変更するためのメソッドも用意されています。

スタブには、無名関数と既存の関数をラップしたものがあります。既存の関数をスタブでラップした場合、元の関数は呼び出されません。
```

うーん、わかるようなわからんような。

この時点で、ざっくり

`mock`
動作を期待するもの。期待する動作するか、確認すべき対象

`sinon`
あらかじめ動作を決めちゃうもの

という理解をしました。

そして、実務コードを読むと、大まかにそんな感じであってそう。

## 結論の理由
mockで期待する動作を検証できるのであれば、基本mockでいいのではと思います。

しかし、テスト対象の中で他のメソッドを呼んでいて、それらの挙動は、それらのテスト層で担保されているとします。

であれば、これらをmockとして挙動の注入を行う必要はないですよね。

そこで、

- テストするまでもない関数
- 意図的に条件分岐の中に入りたい

場合に、「動作を規定できる」stubを使っちゃおうという、最初の結論になるわけです。





