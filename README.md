# DockerでJupyter Notebookを使う

## Tensorflow Notebookイメージを元にTransformers入りのイメージを作る。

```
FROM jupyter/tensorflow-notebook

USER $NB_UID

RUN pip install --upgrade pip && \
    pip install transformers && \
    fix-permissions "/home/${NB_USER}"
```

## Docker composeファイル

```
services:
  transformers-notebook:
    build: ./
    ports:
      - 9888:8888
    environment:
      - JUPYTER_TOKEN=トークンまたはパスワード
    volumes:
      - ./:/home/jovyan
```

## 実行

```
$ docker compose up
```

## pip installでターゲットを見つけられないというエラーが出た場合

次のようなエラーが出ることがあります。

```
Failed to establish a new connection: [Errno -2] Name or service not known',)': ??????
```

これはDockerがネットワーク上の名前解決ができないことに起因します。その場合はDockerサービスの設定ファイル /usr/lib/systemd/system/docker.service を編集して、Dockerが参照するDNSのアドレスを教えてあげます。

### Dockerサービスの設定ファイルを編集

```
$ sudo nvim /usr/lib/systemd/system/docker.service
```
[編集前]
```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

[編集後]
```
ExecStart=/usr/bin/dockerd -H fd:// --dns 192.168.0.1 --dns 8.8.8.8 --containerd=/run/containerd/containerd.sock
```
### Dockerサービスを再起動

```
$ sudo systemctl daemon-reload

$ sudo service docker restart
```


