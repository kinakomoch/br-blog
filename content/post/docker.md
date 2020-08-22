---
title: "Dockerfile, docker-composeの設定について"
description: ""
date: "2020-08-23T00:03:01+09:00"
thumbnail: ""
categories:
  - ""
tags:
  - "Docker"
---

# Dockerfile, docker-composeの設定方法

Dockerfileには通常一つのコンテンツに関するものだけを記述する

例) pythonと同時にpipでpackageもインストールする場合

Dockerfile
```dockerfile
FROM python:latest

COPY requirements.txt requirements.txt

RUN pip install --upgrade pip setuptools \
 && pip install -r requirements.txt
```

"FROM"でベースとなるイメージをDockerHubから取得。セキュリティの観点からなるべくOfficial imageを取得するべき。

"COPY"はhost(自分が実際に利用しているPC)のrequirements.txtをDocker内のrequirements.txtにコピーしている

"RUN"はdockerが立ち上がる際に実行されるコマンド群を記述する。中間イメージをなるべく作らない(重い処理をさせない)ために"RUN"は&&を利用してできるだけ一度で済ませる

requirements.txt
```
matplotlib==3.3.1
mysql-connector
```

上の例ではdockerfileでpythonをインストールしている。
その際にpipを利用してモジュールをインストールしているが、別ファイルのrequirements.txtにモジュールとversionを指定しておく。-rオプションを使うことでrequirements.txtに書かれたモジュールをまとめてインストールすることができる。ここでモジュールのversionは指定しなくても良い。

次にdocker-compose.ymlについて説明する