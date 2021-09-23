---
title: "Vscodeをインストールしたら最初にやること"
date: 2021-09-23T16:27:59+09:00
draft: false
---

VSCodeをインストールしたら最初に入れておくべきプラグインです。  
これがないと仕事にならない
<!--more-->

## Japanese Language Pack
日本人なので

## Docker
バックエンドエンジニアなので。  
あんまり使うことないけど、人に教えるときにあると便利

## Go
Gopherなので

## Vutur
Vue書くので

## Gitlends
差分管理をしやすくする、blameしやすくするため

## Prettier
フォーマット作業は人間の仕事ではないので

## PlantUML
UMLの図を書くため

## Add jsdoc comments
jsdocのコメント表記はなるべく自動で生成しておきたい

## その他の設定
### コピペするならインデントを維持したい
`setttings.json`に下記を追記する
```
"editor.formatOnPaste": true,
```
