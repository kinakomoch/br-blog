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

### Dockerfileには通常一つのコンテンツに関するものだけを記述する

例) jupyterと同時にpipでpackageもインストールする場合

Dockerfile
```dockerfile
FROM jupyter/datascience-notebook

COPY requirements.txt requirements.txt

RUN pip install --upgrade pip setuptools \
 && pip install -r requirements.txt
```

"FROM"でベースとなるイメージをDockerHubから取得。セキュリティの観点からなるべくOfficial imageを取得するべき。

"COPY"はhost(自分が実際に利用しているPC)のrequirements.txtをDocker内のrequirements.txtにコピーしている。

"RUN"はdockerが立ち上がる際に実行されるコマンド群を記述する。中間イメージをなるべく作らない(重い処理をさせない)ために"RUN"は&&を利用してできるだけ一度で済ませる。

注意点としてpipでインターネットを経由してファイルを取得するようなものの場合、取得できない場合がある。その際はDNSの設定ができてない可能性がある。その際は以下のファイルを作成してdockerを再起動させることでうまくいく場合がある。
最初のアドレスは自分の環境に合わせて修正すると良い。

/etc/docker/daemon.json
```json
{
  "dns": ["192.168.26.2", "8.8.8.8"]
}
```

requirements.txt
```
matplotlib==3.3.1
mysql-connector
```

上の例ではdockerfileでpythonをインストールしている。
その際にpipを利用してモジュールをインストールしているが、別ファイルのrequirements.txtにモジュールとversionを指定しておく。-rオプションを使うことでrequirements.txtに書かれたモジュールをまとめてインストールすることができる。ここでモジュールのversionは指定しなくても良い。

次にdocker-compose.ymlについて説明する

ここではディレクトリ構造を以下に示しておく。

```
└── app
    ├── jupyterlab
    │   ├── requirements.txt
    │   └── Dockerfile
    ├── db
    └── docker-compose.yml
```

docker-compose.yml
```dockerfile
version: "3.8"
services:

    jupyterlab:
        build: .
        environment:
            - JUPYTER_ENABLE_LAB=yes
        ports:
            - "8888:8888"
         command: start-notebook.sh --NotebookApp.token=''
    
    db:
        image: mariadb:latest
         environment:
          - MYSQL_ROOT_PASSWORD=example
          - MYSQL_DATABASE=example
          - MYSQL_USER=example
          - MYSQL_PASSWORD=example
        ports:
            - "3307:3306"
        volumes:
            - "./db:/var/lib/mysql"
```

buildはDockerfileがある場合<br>
imageはDocker imageを取得する場合にそれぞれ利用する。どちらかは必ず利用する。

environmentはDockerの環境変数などの設定を行う。

portsは <hostのport>:<dockerのport>で指定する。

volumesもportsと似たような感じで <hostのディレクトリ>:<dockerのディレクトリ>で指定する。

linkは~~container間を接続~~　現在は自動でリンクを作るので不要らしい。

commandはデフォルトのコマンドの上書き

他にもuserを設定したりなんかができる。

以上でおおよそのDockerfileとdocker-compose.ymlの書き方についてまとめとする。

(Dockerの勉強で参考にしたサイト)
 - https://y-ohgi.com/introduction-docker/
 - https://stackoverflow.com/questions/28668180/cant-install-pip-packages-inside-a-docker-container-with-ubuntu
 