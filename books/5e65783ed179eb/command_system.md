---
title: "コマンドシステム"
free: false
---

本章ではコマンドシステムについて解説します。

## Dragonflyのコマンドシステム

Dragonflyでは構造体を使って宣言的にコマンドの引数を定義できます。
引数の種類や順序によって機能を別けるコマンドを作成する場合でも、引数のセット毎に構造体を定義・登録しプログラムの複雑化を防ぐことができます。

## コマンド入門

### コマンドの引数と実行時の処理を定義する

まずは実行者にメッセージを表示するHelloWorldCommandを定義しましょう。
Dragonflyのコマンドシステムにコマンドを登録するにはRunnableインターフェースを実装する必要があります。Runnableインターフェースを実装して実行時の処理を定義しましょう。

:::message
Dragonfly公式ではこの構造体の名称について言及されていませんが、コマンドの引数の並び型によって構造体を作成するため、便宜上この構造体のことを「コマンドの引数セット用の構造体」と呼びます。
:::

```go
// ① コマンドの引数セット用の構造体を定義する
// 何も引数を取らないコマンド
type HelloWorldCommand struct {}

// ② Runnableを実装し、実行時の処理を定義する
// 使わない引数は_で宣言します。変数・レシーバも同様です
func (c HelloWorldCommand) Run(_ cmd.Source, o *cmd.Output) {
	// コマンドの実行者にメッセージを表示する
	o.Printf("Hello, World!")
}
```

以上でコマンドの定義が完了しました。

### コマンドを登録する

定義したコマンドをDragonflyに登録します。

```go
import "github.com/df-mc/dragonfly/server/cmd"

func main() {
	// 省略
	
	// コマンドを登録する
	cmd.Register(cmd.New("helloworld", `Show "Hello, World!"`, nil, HelloWorldCommand{}))
}
```

cmd.New()でコマンドを定義します。第一引数から順にコマンド名、説明、エイリアス、引数を定義した構造体のインスタンスを渡します。
cmd.Register()で定義したコマンドをDragonflyに登録し、サーバー上で使えるようにします。

コマンドにエイリアスを追加するには、第三引数にエイリアスの配列を渡します。

```go
// コマンドを登録する
cmd.Register(cmd.New("helloworld", "Show \"Hello, World!\"", []string{"alias1", "alias2"}, HelloWorldCommand{}))
```

### コマンドに引数を追加する

コマンドの引数セット用の構造体にフィールドを追加することで、コマンドの引数を定義することができます。
publicなフィールドの並び順がそのままコマンドの引数の並び順となります。

```go
import "github.com/df-mc/dragonfly/server/cmd"

// ① コマンドの引数セット用の構造体を定義する
type HelloWorldCommand struct {
	// ③ 引数を追加する
	// フィールド名の頭文字を大文字にすることで、そのフィールドのアクセス可能範囲をpublicにすることができる
	// フィールドの末尾にある`(バッククオート)で囲まれた文字列はタグと呼ばれるもの。動作に直接影響するものではなく、構造体にメタ情報を追加する際に使われる
	// フォーマットは`タグのキー:"タグの値"`
	Hoge string `cmd:"hoge"`
	
	// 複数の引数を取ることもできる
	Fuga int `cmd:"fuga"`
}

// ② Runnableを実装し、実行時の処理を定義する
func (c HelloWorldCommand) Run(_ cmd.Source, o *cmd.Output) {
	// コマンドの実行者にメッセージを表示する
	o.Printf("Hello, World!")
	// コマンドの引数をコマンドの実行者に表示する
	o.Printf("Hoge: %s, Fuga: %s", c.Hoge, c.Fuga)
}
```

以上はstring型のHogeとint型のFugaの二つの引数を定義した例です。実行時にRun関数のレシーバにコマンドの引数がフィールドに代入されたインスタンスが渡されます。
フィールドに定義している`cmd`をキーにしたタグはコマンドのサジェストに使われます。上記の例では、`/helloworld`と入力すると続けて`<hoge: string> <fuga: int>`とサジェストされます。

引数に利用できる型は

- 符号付整数(`int`, `int8`, `int16`, `int32`, `int64`)
- 符号なし整数(`uint`, `uint8`, `uint16`, `uint32`, `uint64`)
- 浮動点少数(`float32`, `float64`)
- 文字列(`strin`)
- 真偽値(`bool`)
- 三次元(`mgl64.Vec3`)
- 文字列の配列(`cmd.Varargs`)
- ターゲットのスライス(`[]cmd.Target`)
- サブコマンド(`cmd.SubCommand`)
- オプショナル(`cmd.Optional[T]`)
- 列挙型([`cmd.Enum`](https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/parameter.go#L30-L40)インターフェース)
- [`cmd.Parameter`](https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/parameter.go#L11-L18)インターフェースを実装した任意の型

です。

プリミティブ型はもちろんのこと、マインクラフト上で必要な型は一通り揃えられています。独自の構造体にも`cmdParameter`インターフェースを実装すればコマンドの引数として扱うことができます。

:::message alert
文字列の配列とオプショナルの性質上、**構造体にオプショナルフィールドと文字列の配列フィールドを共存させることはできません**。
:::

:::message alert
各インターフェースで定義されている`Type()`メソッドで引数名を定義しても、クライアント側で正しく表示されないバグがあります。
:::

#### 文字列の配列(`cmd.Varargs`)

文字列の配列を使う場合は、構造体の最後のフィールドに指定する必要があります。
文字列の配列と称していますが、cmd.Varargsの実態はstringです。「 」(半角空白)で文字列を分割する必要があります。

#### サブコマンド(`cmd.SubCommand`)

`cmd.SubCommand`をフィールドに定義すると、サブコマンド用の識別子を実装することができます。

```go
import "github.com/df-mc/dragonfly/server/cmd"

type HogeCommand struct {
	// フィールド名が識別子になる
	Hoge cmd.SubCommand
}
```

上記の例では、`/コマンド Hoge`で`HogeCommand`を実行することができます。

サブコマンド名を自由に設定したい場合は、フィールドに`cmd`をキーとするタグを追加します。

```go
import "github.com/df-mc/dragonfly/server/cmd"

type HogeCommand struct {
	// サブコマンド名をタグで指定する
	// タグの値がサブコマンド名となる
	Hoge cmd.SubCommand `cmd:"fuga"`
}
```

`/コマンド fuga`で`HogeCommand`を実行することができます。

#### オプショナル(`cmd.Optional[T]`)

`cmd.Optional[T]`でフィールドに引数を定義すると、その引数を指定するかしないかをユーザー側が自由に選ぶことができます。
オプショナルを使う場合は、構造体の最後のフィールドに指定する必要があります。

```go
import "github.com/df-mc/dragonfly/server/cmd"

type GiveCommand struct {
	Item string
	Amount cmd.Optional[int]
}
```

### 列挙型(`cmd.Enum`)

https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/parameter.go#L30-L40

列挙型は、ユーザーに選択肢を表示し、その中の一つを選ばせる場合に使います。
定義した列挙型は、構造体のフィールドに定義することで引数として使えます。

```go
import "github.com/df-mc/dragonfly/server/cmd"

// 選択肢のための型を定義
// 文字列の選択肢で受け取るためstringを使用する
type FoodEnum string

// タグを省略した場合に使われる引数名を返す
func (e FoodEnum) Type() string {
	return "Food"
}

// 選択肢を文字列の配列で返す
func (e FoodEnum) Options(_ cmd.Source) []string {
	return string[]{"apple", "bread", "carrot", "Cooked Porkchop"}
}

type FoodCommand struct {
	Food FoodEnum
}

```

#### `cmd.Parameter`インターフェースを実装した任意の型

https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/parameter.go#L11-L18

Dragonflyが用意している引数の型以外にも、対象の型に`cmd.Parameter`インターフェースを実装することで引数として扱えるようになります。

`int`用に実装されている`parser#parse()`メソッドを見てみましょう。

https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/argument.go#L126-L137

`line#Next()`次に処理される引数(文字列)を受け取り、その値を`strconv.ParseInt()`でint型に変換しています。最後に`v.SetInt()`で引数に対応する値をセットしています。
処理の途中でエラーが発生した場合は、そのまま関数の戻り値にエラーをセットし、パースが失敗したこと示します。

同じように対象の型に`cmd.Parameter`インターフェースを実装すれば、送信されたコマンドを正しくパースできます。

```go
import "github.com/df-mc/dragonfly/server/cmd"

// Address型を定義する
type Address struct {
	Hostname string
	Port     uint16
}

// 引数名を定義する
func (_ Address) Type() string {
	return "address"
}

// 引数のパーサーを定義する
// パースに失敗した場合はnil以外を返す
func (_ Address) Parse(line *cmd.Line, v reflect.Value) error {
	// 引数を取得する
	arg, ok := line.Next()

	if !ok {
		// 引数がなかった場合はエラーを返す
		return cmd.ErrInsufficientArgs
	}

	// :でアドレスをホストとポートに分割
	hostAndPort := strings.Split(arg, ":")
	if len(hostAndPort) != 2 {
		return fmt.Errorf(`cannot parse argument "%v" as type %v`, arg, v.Kind())
	}

	// ホストとポートを抽出

	h := hostAndPort[0]
	// ポート番号をuint16に変換
	p, error := strconv.ParseUint(hostAndPort[1], 10, 16)

	if error != nil {
		return fmt.Errorf(`cannot parse argument "%v" as type %v`, arg, v.Kind())
	}

	// 構造体をセットする
	v.Set(reflect.ValueOf(Address{Hostname: h, Port: uint16(p)}))
	return nil
}

type AddressCommand struct {
	Address Address
}

func (c AddressCommand) Run(_ cmd.Source, o *cmd.Output) {
	o.Printf("hostname: %v, port: %v", c.Address.Hostname, c.Address.Port)
}

func main() {
	// 省略

	cmd.Register(cmd.New("address", "", nil, handler.AddressCommand{}))
	
	srv.Listen()
	for srv.Accept(nil) {
	}
}

```

実行例

![](https://storage.googleapis.com/zenn-user-upload/b4b1b5b390d6-20221107.png)
![](https://storage.googleapis.com/zenn-user-upload/ed6856068e3f-20221107.png)

### コマンドの実行に制限を設ける

https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/command.go#L35-L39

`cmd`パッケージには`Allower`インターフェースがあります。このインターフェースをコマンドの引数セット用の構造体に実装することで、引数セットで実行の許可を制御することができます。

```go
import "github.com/df-mc/dragonfly/server/cmd"

func (c HogeCommand) Allow(s cmd.Source) bool {
	// 許可するかどうかを真偽値で返す
}

```

許可する場合は`true`、許可しない場合は`false`を返します。
`false`を返した場合はその引数セットはクライアント側で表示されません。

## コマンド応用
### `cmd.Source`・`cmd.Target`を使う
`Run`メソッドの引数にはコマンドの実行者を表す`cmd.Source`インターフェースを実装した構造体のポインタが渡されます。

https://github.com/df-mc/dragonfly/blob/92efc5c32ddc1b283a5fe8acc6d9966371772ebd/server/cmd/source.go#L8-L16

Dragonflyの場合は現時点でコンソールからの入力ができず、`cmd.Target`型が埋め込まれていることから`cmd.Source`はほぼプレイヤーを表しています。より具体的なプレイヤーの情報へアクセスするためは`cmd.Source`を`*player.Player`として扱う必要があります。`*player.Player`へキャストできるかどうかチェックしてから処理を行いましょう。

```go
import "github.com/df-mc/dragonfly/server/cmd"

func (c HogeCommand) Run(s cmd.Source, o *cmd.Output) {
	// *player.Playerにキャストする
	p, ok := s.(*player.Player)
	if !ok {
		// キャスト失敗
		o.Error("Execute as a player")
	}
	
	// プレイヤーに挨拶する
	o.Print("Hello, ", p.Name(), "!")
}

func (c HogeCommand) Allow(s cmd.Source) bool {
	// *player.Playerにキャストする
	p, ok := s.(*player.Player)
	if !ok {
		// キャスト失敗
		o.Error("Execute as a player")
	}
	
	if p.Name() == "UramnOIL" {
		return true
	} else {
		return false
	}
}

```

おなじく`cmd.Target`インターフェースから構造体のポインタを得る場合にはキャストが必要です。

```go
import "github.com/df-mc/dragonfly/server/cmd"

// ターゲットをプレイヤーのみに絞る

type HogeCommand struct {
	Targets []cmd.Target
}

func (c HogeCommand) Run(_ cmd.Source, o *cmd.Output) {
	// Targetsを*player.Playerの配列にキャストする
	var players []*player.Player
	for _, t := range c.Targets {
		p, ok := t.(*player.Player)
		if !ok {
			// キャスト失敗
			o.Error("プレイヤー以外を指定することはできません")
		}
		players = append(players, p)
	}
	
	// ターゲット全員に挨拶する
	for _, t := range players {
		p.Message("Hello, ", p.Name(), "!")
	}
}

```

### 引数セットを追加する

これまで一つのコマンドに一つの構造体を定義し単体の機能のみを持たせていましたが、実際のコマンドには**一つのコマンドに複数のサブコマンドや引数の型**を指定することがあります。
例えばteleportコマンドでは行き先に座標またはターゲットを指定することができますし、複数のサブコマンドを使って一つのコマンドに複数の機能を持たせる場合があり、ユーザーに価値を届けるために直観的で柔軟な表現を実現する必要があります。

従来のサーバーソフトウェアでは、一つのコマンドに対して一つの実行用のメソッドがあり、コマンドの引数は配列で受け取っていました。
処理の内容や正しい使い方かどうかに関係なくユーザーから送られてくるすべての引数を文字列の配列で受け取るため、**複雑な引数を持つコマンドの検証や処理の分岐は実装者の責務**でした。

以下はPocketMine-MPのコマンドの実装の例です。

```php
public function execute(CommandSender $sender, string $commandLabel, array $args) : bool {
	// $argsの引数の数や内容で処理を分岐させる
	if (count($args) == 0) {
		return false;
	}
	
	switch($arg[0]) {
	case "hoge":
		// 処理1
		break;
	case "fuga":
		// 処理2
		break;
	}
	case "piyo":
		// 処理3
		break;
	
	return true;
}
```

初めに言った通り、Dragonflyでは宣言的に引数を定義するため引数の並びによる処理の分岐や引数の型の検証が一切必要ありません。複数の引数のセットがある場合には自動で分岐するため、プログラムのシンプルさと表現の柔軟性を保ちながら実装することができます。
引数のセットを追加する場合は新たにコマンドの引数セット用の構造体を宣言し、構造体のインスタンスをコマンドの登録時に渡すだけで実現することができます。

```go
import "github.com/df-mc/dragonfly/server/cmd"

// タイマーコマンドを実装する
// 開始・終了の実行をサブコマンドで分ける

type TimerStartCommand struct {
	Start cmd.SubCommand `cmd:"start"`
}

func (c TimerStartCommand) Run(cmd.Source, *cmd.Output) {
	// `/timer start`で実行される処理
}

type TimerStopCommand struct {
	Stop cmd.SubCommand `cmd:"stop"`
}

func (c TimerStopCommand) Run(cmd.Source, *cmd.Output) {
	// `/timer stop`で実行される処理
}

// timerコマンドを登録する
// 第四引数はcmd.Runnableの可変長引数となっており、引数のセットを追加するにはカンマ区切りで引数の末尾にcmd.Runnableを追加します。
cmd.Register("timer", "measure time", nil, TimerStartCommand{}, TimerStopCommand{})
```

```go
import "github.com/df-mc/dragonfly/server/cmd"

// テレポートコマンドを実装する
// コマンドの第一引数をターゲットまたは座標で指定できるようにする

// ターゲットで指定する場合
type TeleportToTargetCommand struct {
	Target []cmd.Target `cmd:"destination"`
}

func (c TeleportToTargetCommand) Run(cmd.Source, *cmd.Output) {
	// `/teleport @r`等で実行される処理
}

// 座標で指定する場合
type TeleportToCoordCommand struct {
	Coord mgl64.Vec3 `cmd:"destination"`
}

func (c TeleportToCoordCommand) Run(cmd.Source, *cmd.Output) {
	// `/timer 0 100 0`等で実行される処理
}

// teleportコマンドを登録する
cmd.Register("teleport", "teleport to destination", nil, TeleportToTargetCommand{}, TeleportToCoordCommand{})

```

---

以上のように、単純に引数のセットを定義した`cmd.Runnable`を実装した構造体を増やしていくだけで機能を追加することができます。

:::message alert
送信された引数による分岐は、登録された`cmd.Runnable`の中で一番初めに検証をパスした引数セット用の構造体が実行される仕組みであるため、引数にすべての文字列を受け取る`string`を使用すると意図しない処理が実行される可能性があります。
:::

```go
import "github.com/df-mc/dragonfly/server/cmd"

// 検証が緩いstring
type Hoge1Command struct {
	Hoge1 string
}

func (c Hoge1Command) Run(_ cmd.Source, o *cmd.Output) {
	o.Print("string")
}

// 検証が厳しいint
type Hoge2Command struct {
	Hoge2 int
}

func (c Hoge2Command) Run(_ cmd.Source, o *cmd.Output) {
	o.Print("int")
}

cmd.Register(cmd.New("hoge", "", nil, Hoge1Command{}, Hoge2Command{})
```

上記の例は、第一引数に`string`か`int`をとるコマンドを登録しています。
しかし、第一引数にintを渡そうとして`/hoge 100`を実行しても、プレイヤーには「string」と表示されてしまいます。

![](https://storage.googleapis.com/zenn-user-upload/a3a4502e011c-20221108.png)
![](https://storage.googleapis.com/zenn-user-upload/d77c4e104bc1-20221108.png)

