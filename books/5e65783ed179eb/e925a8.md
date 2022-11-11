---
title: "起動してみる"
free: false
---

まず初めに、Dragonflyを起動してみましょう。

## 環境

以下は私が使用している環境です。
```
OS: Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-52-generic x86_64)
Go: go version go1.19.2 linux/amd64
Git: git version 2.38.1
```

DragonflyはGo言語で書かれているため、ソースコードからの起動やバイナリの生成にはGo言語の環境が必要になります。Go言語はクロスプラットフォームでの開発が可能となっており、実行環境を選びません。当書ではLinux(Ubuntu 22.04.1 LTS)を前提に開発を進めていきますが、**基本的にどのPCでも問題なく動作します**。

Dragonflyが必要とするGoのバージョンは**1.18以上**です。Goをインストールしていない、または1.18未満のバージョンを使用している場合は https://go.dev/doc/install からGoをインストールしてください。

## Dragonflyを起動する
### STEP1 リポジトリをクローン
https://github.com/df-mc/dragonfly からリポジトリをクローンします。
クローン後はローカルリポジトリに移動します。

```sh
git clone https://github.com/df-mc/dragonfly.git
cd dragonfly
```

### STEP2 実行
ソースコードから直接Dragonflyを起動させます。

https://github.com/df-mc/dragonfly/blob/master/main.go

```sh
go run main.go
```

`go run main.go`で`main.go`に記述されたDragonflyのプログラムが起動します。起動が完了すると`Server running on [::]:19132`という文字列がコンソールに表示されるとともに、Dragonfly用の設定ファイルやセーブデータがカレントディレクトリに生成されます。

`ctrl + c`を入力するとサーバーが安全に終了します。

---

特に難しい操作をすることなくDragonflyを起動することができます。
これでクライアントでサーバーに接続することができるようになりましたが、コマンドのサジェストを見てもらえるとわかる通り、ヘルプコマンド以外何一つ登録されておらず、**サーバーソフト単体では特にできることはありません**。
これがDragonflyが目指すシンプルさの一つであり、この最低限の状態から機能を追加し、自分専用のサーバーを作成していきます。