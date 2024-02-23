---
title: Golang 用デバッガ delve のコマンド紹介
tags:
  - Go
  - delve
private: false
updated_at: '2024-02-24T00:18:40+09:00'
id: 9f2fbc67d84d85bbe3a2
organization_url_name: null
slide: false
ignorePublish: false
---

Golang 用のデバッガ _delve_ についてお話します。

# What is delve

_[delve](https://github.com/go-delve/delve)_ とは、Golang 用のデバッガです。
_Golang_ で作成したアプリケーションに何か不具合があったときに利用する便利ツールです。

- nil ポインタ panic が発生したのが `FindBy()` っぽいんだけど 、実行したとき変数に何がセットされているのか知りたい
- `Execute()` 関数が呼ばれるまでにどういう経路をたどったのか知りたい

といったときにその問題の解決に資するものです。

C 言語でコンパイルした実行ファイルをデバッグする際に _[gdb](https://ja.wikipedia.org/wiki/GNU%E3%83%87%E3%83%90%E3%83%83%E3%82%AC)_ を使ったことがある人、ただの _Golang_ 用の _gdb_ です。
コマンド一覧だけ見て読了していただいて構いません。

# How to install

Check [GitHub](https://github.com/go-delve/delve/tree/master/Documentation/installation).

## Windows
1. `go get -u github.com/go-delve/delve/cmd/dlv` 
※もし `%GOPATH%\bin` を PATH に通していなかったら、通してください。

## MacOS
I'm not a Mac user so please refer to [here](https://github.com/go-delve/delve/blob/master/Documentation/installation/osx/install.md).

# How to use
_main_ パッケージがあるディレクトリ上で `dlv debug` (**delve ではなく dlv であることに注意**）を実行。デバッガが起動し、
以降はインタラクティブに操作していきますので、以下のコマンド一覧をご参考ください。

# コマンド一覧

以下、`コマンド（コマンド略）` の順で一覧さしあげます。

## `break(b)`
ブレークポイントを貼ります。文字の通り、ブレークする場所です。
例えば、`breakpoint main.FindBy` とすると、`main` パッケージの `FindBy()` 関数にブレークポイントを貼ります。
ブレークポイントを張った後、実行を続けると、ブレークポイントが設定された場所まで実行されますので、
デバッグしたい関数が分かっている場合に有用です。

```
(dlv) break sales_aggregation.Execute
Breakpoint 1 set at 0x9d878f for main.Execute() C:/test.go:21
```

## `continue(c)`
ブレークポイントを設定した場所までプログラムを実行します。
`breakpoint` コマンドを使用した後の実行、というのが常になると思います。

```
(dlv) continue
```
※実行時にブレークポイントに到達すると、ブレークポイント周辺のソースコードが表示されます。

## `next(n)`
文字の通り、**次**にいきます。
ここでいう**次**は、**ソースコードの次の行**です。
例えば、以下のようなコードがあり、デバッガが `>` の箇所を実行しているとき、

```go
1: > 	q := fmt.Sprintf(base, strings.Join(valueStrings, ","))
2: 	_, err := execx(repo, q, valueArgs...)
3: 	if err != nil {
4: 		return app_errors.NewDatabaseErr(err.Error())
5: 	}
```

`next(n)` コマンドを実行すると、実行場所は以下のようになります。

```go
1: 	q := fmt.Sprintf(base, strings.Join(valueStrings, ","))
2: >	_, err := execx(repo, q, valueArgs...)
3: 	if err != nil {
4: 		return app_errors.NewDatabaseErr(err.Error())
5: 	}
```

見た通り、次の行に移動しています。
次項で説明する `step(s)` とは意味が異なるので丁寧に説明しました。

## `step(s)`
関数をステップイン実行します。
「イン」のニュアンスをよく汲み取っていただければお分かりかもしれません。
こちらのコマンドは、**関数の内部の処理を一つずつ実行します**。
<br>言い換えれば、`next` は関数内部についてはブラックボックスとみなし、`step` はホワイトボックスとみなします。
なので、関数内部の処理を追いたい場合は `step(s)`、関数内部の処理に関心がない場合は `next(n)` でだいたい合っていると思います。

## `list(ls)`
実行している箇所付近のソースコードを表示します。
    
```go
=>  44: func plus(num1 int, num2 int) int {
    45:         return num1 + num2
    46: }
```

## `locals`
ローカル変数を表示します。

    ```
    (dlv) locals
    db = *database/sql.DB nil
    err = error nil
    ```

## `stepout`
現在実行中の関数から抜けます。
`step` して `stepout` ですね。（`step` に _in_ がついていなくてすみません）

## `exit`
exit します。

はい。

# その他
-  `b` したら _ambigous_ って言われましたがどうしたらいいですか？
パッケージ内で名前が重複していてデバッガが関数を一意に識別出来ていませんので、
インポートパスを指定してください。`ex) github.com/username/golang/rdb.test2.FindBy`
※rdbパッケージ内に `FindBy()` という関数が重複している想定

- スタックトレースを見たいんですが。
`stack(bt)`を実行してください。
スタックトレースって？という方は、アセンブリ言語を勉強してください。すごく理解出来ます。

- 変数の値を書き換えたい。
`set 変数名 値` で書き換えられます。
- もっとキレイにまとまってるサイトはないの？
[Qiitaのこちらの記事](https://qiita.com/minamijoyo/items/4da68467c1c5d94c8cd7#%E3%82%B9%E3%82%BF%E3%83%83%E3%82%AF%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B9%E3%81%AE%E8%A1%A8%E7%A4%BA)がよくまとまっています。
本記事より一覧性が高く見やすいので、ここまで来た方は悔やんでください。

# まとめ
筆者がよく使うコマンドをメインに紹介しました。
習うより慣れろなんで、[Qiitaのこちらの記事](https://qiita.com/minamijoyo/items/4da68467c1c5d94c8cd7#%E3%82%B9%E3%82%BF%E3%83%83%E3%82%AF%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B9%E3%81%AE%E8%A1%A8%E7%A4%BA)を参考にしながら、慣れていきましょう。

