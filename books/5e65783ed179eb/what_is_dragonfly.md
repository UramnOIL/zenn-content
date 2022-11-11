---
title: "Dragonflyとは"
free: false
---

https://github.com/df-mc/dragonfly

:::message
Dragonflyは現在初期開発段階です。
:::

> Dragonfly is a heavily asynchronous server software for Minecraft Bedrock Edition written in Go. It was written with scalability and simplicity in mind and aims to make the process of setting up a server and modifying it easy. Unlike other Minecraft server software, Dragonfly is generally used as a library to extend.

> Dragonfly は、Go で書かれた Minecraft Bedrock Edition 用の重厚な非同期サーバーソフトウェアです。**スケーラビリティとシンプルさ**を念頭に置いて書かれており、サーバーのセットアップと修正のプロセスを簡単にすることを目的としています。他の Minecraft サーバーソフトウェアとは異なり、Dragonfly は一般的に拡張するためのライブラリとして使用されます。(Deeplによる翻訳)

https://github.com/df-mc/dragonfly#dragonfly　より抜粋

READMEに記載されている通り、Go言語のシンプルさに則ったマインクラフト向けのサーバーソフトウェアです。**サーバーを拡張するための最低限の機能のみ搭載されており、標準的なコマンドやパーミッションすら用意されていません**。そのシンプルさ故に拡張するために必要な行数は圧倒的に少なく、またGo言語らしい高速で直感的、モダンなプログラミングができるように設計されています。

他のサーバーソフトウェアと大きく異なる点として、サーバーソフトウェアが**単体で実行が可能なファイルではなく、ライブラリとして提供されています**。プラグ＆プレイなプラグイン機構がなくなり、main関数にサーバーの起動と新規機能のためのプログラムを書く必要がありますが、サーバーソフトウェア直接実行することで**プラグインのデバッグが容易になります**。
そのほかにも、サーバーソフトウェアをライブラリとして提供したことで、サーバーソフトウェアと新規機能をひとまとめにし、**デプロイに適した単一のファイル(シングルバイナリ)を生成することができます**。

Dragonflyはプログラミングに慣れている開発者向けではありますが、開発速度の面で突出している**デベロッパーフレンドリーなサーバーソフトウェア**です。
将来的には、拡張機能をフルスクラッチするような開発速度重視のミニゲームサーバーはDragonfly、だれでも簡単に拡張機能を追加することを重視する生活サーバーはプラグインが豊富な既存のサーバーソフトウェアまたはBDSを拡張する[LiteLoaderBDS](https://github.com/LiteLDev/LiteLoaderBDS)等で利用シーンを分けることができるかもしれません。
