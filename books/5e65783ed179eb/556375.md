---
title: "サーバーを作ってみる"
free: false
---

前章ではDragonfly本体のリポジトリをクローンしてサーバーを動かしていましたが、本来はDragonflyをライブラリとして使用します。Dragonflyのリポジトリのクローンから始めても問題なく動作しますが、余計なコードが混ざることとなるので新規のプロジェクトを開始しましょう。
プロジェクトの初期化からライブラリの追加、`main.go`の作成まで行います。

## プロジェクトの初期化


`go mod init`を使ってプロジェクトを初期化します。[テンプレート](
https://github.com/df-mc/template)を使用する場合は本章を飛ばしても構いません。

```sh
mkdir <プロジェクト名>
cd <プロジェクト名>
go mod init github.com/<ユーザーネーム>/<プロジェクト名>
```

`go mod init`の後に入る文字列は、作成したプロジェクトを`go get`でライブラリとしてインストールするときに使う識別子として使われます。
特に理由がなければGitHubで作成されるプロジェクトのURLに従って上記のコマンドを修正してください。

Git等のVCSを使う場合は合わせて初期化してください。

## ライブラリの追加

```sh
go get -v -u \
	github.com/df-mc/dragonfly \
	github.com/pelletier/go-toml \
	github.com/sirupsen/logrus
```

## メインプログラムの作成

`main.go`ファイルを作成し

https://github.com/df-mc/dragonfly/blob/master/main.go

を丸々コピー&ペーストしてください。プロジェクトを公開する場合はライセンスに注意してください。

## サーバーの起動

`go run main.go`を実行すると、前章と同じくサーバーが起動されます。

---

基本的に`main.go`ファイルにプログラムを追加することでサーバーを拡張していきます。次章からはサーバーの拡張について解説します。