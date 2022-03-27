---
title: "m1マシンにrailsプロジェクトを移管した時に詰まった忘備録"
date: 2022-03-27T17:29:33+09:00
draft: false
---
既存のRailsプロジェクトをM1マシンに移行したところかなりどハマりしたので、同じような方の助けになればと思い、記録を残します。

<!--more-->

`ハマったこと`
1. `docker-compose up` でgemが `could not find xxxxx` になって、appコンテナ作成にコケる
2. gemの`nokogiri`が `ERROR: It looks like you're trying to use Nokogiri as a precompiled native gem on a system with glibc < 2.19:` を吐いて、appコンテナ作成にコケる
3. mysqlのimage pull が `no matching manifest for linux/arm64/v8 in the manifest list entries.` となってできない
4. webpackerが`Webpacker can’t find application.js in /myapp/public/packs/manifest.json` となってアクセス時にエラる
5. `rails db:setup` で `Function not implemented - Failed to initialize inotify (Errno::ENOSYS)`

`対応`
- 前提として、appのimageが巨大なので、不要なデータはキャッシュ含めて消す。作成に失敗した時も容量削減のために必ず消すこと。
  - `docker system prune`
  - `docker rmi xxxx yyyy zzz`
  - `docker builder prune`
1. volumeを消して、cacheなしでbuildする
  - `docker volume ls` => `docker volume rm xxx yyy`
  - `docker-compose build --no-cache`
2. Dockerfileで定義しているrubyのバージョンのスコープを限定する
  - `FROM ruby:2.6.4` =>  `FROM amd64/ruby:2.6.4`
3. platformを指定する
  - `platform=linux/x86_64`
4. manifest.jsを作って下記を記述
5. development.yamlを修正する
```
//= link_tree ../fonts <-自分の環境n合わせること
//= link_directory ../javascripts .js
//= link_directory ../stylesheets .css
```

---
`構成`
- docker-compose.yml
- docker-compose-override.yaml
- Dockerfile.development
を利用

`詳細`
docker-compose.yaml  
基本的な定義のみを記述している。
```
services:
  databse:
    restart: always
    image: mysql:latest
    command: --default-authentication-plugin=mysql_native_password
    ....
  redis:
    image: redis:latest
    volumes:
      - ./tmp/redis:/data
volumes:
  mysql-datavolume:
    driver: local
  bundle:
    driver: local
```

docker-compose-override.yaml
app定義を記述
```
app:
  ports:
    - 3000:3000
  build:
    context: .
    dockerfile: Dockerfile.development
  command: xxxx
  ....
database:
    ports:
      - xxxxx
  redis:
    ports:
      - xxxx

```

Dockerfile.development
stepを定義
```
FROM node:14 as node
FROM ruby:2.6.4
...
```

----
## それぞれの原因
### 1. gem の `could not find xxx` 問題
cacheが残っていると Dockerfile.development の bundle install step を上手く実行してくれない。

volumeを消して再度buildすれば問題ない。  
ただし、imageが巨大な場合はtimeoutするかもなので、docker-compose up で一気にやるのは良くなくて、mysql や redis などの image 提供元があるものに関しては skip したいので、`docker-compose build --no-cache` で build のみした方がいい。

railsもimageに詰めようと思ったが、さらに巨大になるのは(含めなくても15GB)嫌なので、アプリは外で動かすことにしている。

### 2. nokogiri の `glibc < 2.19` 問題
要求されているバージョンとホストOSが提供しているバージョンがずれていることが問題。
取得するruby image のスコープを明示的に amd64/に絞ることでOK

### 3. mysql の image pull 問題
シンプルにarmが提供されてないから。  
これだけの理由でmeriaDBにするのは意味ないので、platformを指定して凌ぐこと。

### 4. webpacker の could not find application/xxx 問題
根幹の原因は調査できていない。  
とりあえず作れば動くが本質ではないので、何か知っている方いましたら教えていただきたいです。

### 5. 
issueが上がっていた。
https://github.com/evilmartians/terraforming-rails/issues/34#issuecomment-872021786

下記に修正すれば良さそう
`config.file_watcher = ActiveSupport::FileUpdateChecker`

---

## 実行の手順
1. bundle exec rails s
2. docker-compose up
3. (別sessionで) bundle exec rails db:setup
---
