---
title: "Pythonでlambda実行したらハマった"
date: 2022-04-16T13:18:14+09:00
draft: false
---

Pythonで AWS lambda を実行しようとしたらいくつかハマったので、その時の忘備録

<!--more-->
`ハマったこと`
もちろんローカルでは動作している前提です。

1. 実行時に `"Unable to import module '...': No module named ..."` が出て実行できない
2. 各種ライブラリがimportできない
3. `lxml` が `Couldn't find a tree builder with the features you requested: lxml. Do you need to install a parser library?` で import できない
4. ファイル書き込みが `Read Only file xxxx` でできない

---
## 対処法
1. 実行ファイル、関数名を設定値と合わせる
当たり前ですが、lambda側の設定とファイル・関数を合わせる必要があります。  
初期では、「lambda_function」の「lambda_handler」を実行するようになっているので、下記構成にすると動くでしょう。

```
lambda_function.py

  def lambda_handler(event, context):
    some impl....
```

2. 実行ファイルと同階層・かつupload時にはフォルダの「中身」をzipに固める必要がある。
まず、サードパーティのライブラリを使う場合、実行ファイルと同階層に展開する必要があります。
```
pip install xxx -t .
```
これでルートに展開されます。  

そして、「この全体を」zipに固める必要があります。

つまり、lambda_function.py と 各種ライブラリが同階層に入った「フォルダ本体」をzipにするのではなく、「これら全体」を選択して固めます。

mac環境であれば、「アーカイブ.zip」というファイルができるでしょう。  

アップロードするのはこれです。

親フォルダを固めたzipではありません。

3. 標準を使う
調べていると同じような症状が出ている人が多く、特に lxml にこだわる理由もないのであれば、`html5lib` に切り替えるといいでしょう。

4. `/tmp/` に作成する
lambdaでは書き込みできるのは `/tmp/` 配下に限定されるようです。
なので、ファイル書き込みしたい、というのであればここに移すのがいいです。  

自分の場合はこのようなアプローチを取りました。
- `/tmp` 配下に必要なフォルダを `os.mkdir` で作成
- write権限で必要なファイルを `/tmp` 配下でopen
  - そのファイル名は日付をsuffixにつけることで、実行プロセスが対象を一意に特定できるようにする
- 最後に `/tmp` 配下を `rm` することでクリーンする

---

## まとめ
lambdaをpythonで使ったことがなかったので、モジュールの展開などで混乱しました。  
特に仮想環境を使っている人がほとんどだと思うので、lambda環境特有の仕様に合わせるために混乱しちゃうかもしれません。  


