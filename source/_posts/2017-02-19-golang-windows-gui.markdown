---
layout: post
title: "Go言語でWindowsのnetshラッパーのGUIアプリを作った"
date: 2017-02-19 19:00:00 +0900
comments: true
categories: golang windows
---
インテリジェントスイッチングハブなどのネットワーク機器の設定をする時に、
Windows だといちいちコントロールパネルを辿ってアダプター一覧を表示して、
そこから UAC の権限昇格 (環境によってはパスワード入力が必要) を挟んでプロパティを開いて、
IPv4 の設定を開いて変更して閉じて行って反映、という作業が大変だったので、
`netsh` を呼び出して[省力化する GUI アプリ](https://github.com/znz/winipset)を作りました。

<!--more-->

## 環境

- 開発環境: go version go1.7.5 darwin/amd64 (Homebrew で入れた go)
- 動作対象環境: Windows 7 などの Windows 環境

## GUI ツールキットの選定

開発環境を Windows に入れたくなかった (配布先のユーザーの環境に近い環境にしておきたかった) のと、確実に1バイナリで配布がすみそうということで、
最終的には [WindowsでGo言語でGUIするにはWALKがいいかもしれない](http://qiita.com/alucky0707/items/b066ccd2ff8517cf79fb) の記事をみて [A Windows GUI toolkit for the Go Programming Language (WALK)](https://github.com/lxn/walk) に決めました。

## go get walk

現状は https://github.com/lxn/walk/issues/237 に報告されているように

    %  GOPATH=/tmp/g GOOS=windows go get github.com/lxn/walk
    # github.com/lxn/walk
    /tmp/g/src/github.com/lxn/walk/splitterlayout.go:314: undefined: sort.SliceStable

というエラーになるので、

    % cd $GOPATH/src/github.com/lxn/walk
	% git log -p master

で `sort.SliceStable` が入った直前のコミットを探して、

    % git checkout 5c627b7fa8fb66c201b0273609c61c8117e45bb0
    % cd -
	%  GOPATH=/tmp/g GOOS=windows go get github.com/lxn/walk

のようにちょっと古い WALK を使っています。

## サンプルなどで動作確認

Qiita の記事の例を動かして見たり、`github.com/lxn/walk` の `examples` を参考にしてどんな感じなのか確かめました。

README などに書いてあるように `-ldflags="-H windowsgui"` をつけて、

    GOOS=windows go build -ldflags="-H windowsgui"

でコマンドプロンプトの出ない GUI アプリが作成できました。

## manifest 作成

walk の README には

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
        <assemblyIdentity version="1.0.0.0" processorArchitecture="*" name="SomeFunkyNameHere" type="win32"/>
        <dependency>
            <dependentAssembly>
                <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls" version="6.0.0.0" processorArchitecture="*" publicKeyToken="6595b64144ccf1df" language="*"/>
            </dependentAssembly>
        </dependency>
    </assembly>
```

と書いてありますが、

`examples` 以下から dpiAware の設定も入っている manifest をコピーしてくるのが良さそうです。
コピーしてくると BOM もついているようです。

`name="SomeFunkyNameHere"` は書き換えた方が良いのかと思ったのですが、
`examples` 以下では全て同じだったので、別に書き換えなくても良いようです。

## manifest 変更

これは後でやったことなのですが、
`netsh` で IP アドレスなどの設定を変更するには、
ローカルの Network Configuration Operators のグループの権限が必要なので、
[常に管理者としてアプリケーションを実行させるには？](http://www.atmarkit.co.jp/fdotnet/dotnettips/958uacmanifest/uacmanifest.html)
を参考にして、
自動で UAC の要求をするように以下の設定を追加しました。

```
	<trustInfo xmlns="urn:schemas-microsoft-com:asm.v2">
		<security>
			<requestedPrivileges xmlns="urn:schemas-microsoft-com:asm.v3">
				<requestedExecutionLevel level="highestAvailable" uiAccess="false" />
			</requestedPrivileges>
		</security>
	</trustInfo>
```

これでドメイン環境で Administrators 権限が得られなくても、
自分のアカウントがローカルの Network Configuration Operators グループに所属していれば、
UAC で自分のアカウントのパスワードを入力すれば IP アドレスの変更ができるようになります。
(変更ができるのはコントロールパネルの方からでも `netsh` (を使うこのアプリ) でも。)

## manifest のコンパイル

`rsrc` を実行するのはビルド側なので GOOS の指定は不要です。

    % go get github.com/akavel/rsrc
    % $GOPATH/bin/rsrc -manifest test.manifest -o rsrc.syso
    Manifest ID:  1

`rsrc.syso` がある状態で

    GOOS=windows go build -ldflags="-H windowsgui"

すると作成される `exe` ファイルのサイズがちょっと増えていました。

`rsrc.syso` ファイルを取り込ませる必要があるので、
`go build test.go` などのようにファイル指定での `go build` は避ける必要があるようです。

[Go言語でGUIプログラム on Windows - はけの徒然日記](http://d.hatena.ne.jp/hake/20150816/p1) によると
`-ldflags="-s -H windowsgui"` で strip をかけてサイズを縮小するという方法もあるようです。

## netsh の実行

[WindowsのnetshコマンドでTCP/IPのパラメータを設定する](http://www.atmarkit.co.jp/ait/articles/1002/05/news097.html)
などを参考にすると以下のコマンドを使えば良いことがわかります。

- `netsh interface ip show interfaces`
- `netsh interface ip set address "ローカル エリア接続" dhcp`
- `netsh interface ip set address "ローカル エリア接続" static 192.168.20.2 255.255.255.0`

`show interfaces` で一覧を取得して、
`dhcp` で DHCP に戻す、
`static` で固定 IP アドレス設定です。

ルーターやインテリジェントスイッチングハブに直結して設定する用途を想定しているので、
ネットマスクは `255.255.255.0` 固定で、ゲートウェイなどは設定しません。

まず、Go 言語から呼び出す前に、管理者権限のコマンドプロンプトで実行して期待する動作をするのを確認しました。

ただし、DHCP に戻すのは、なぜかイーサネットケーブルを接続した状態じゃないとエラーになって戻せませんでした。
コントロールパネルだと戻せるので謎挙動です。

## ログ表示

`examples/logview` が便利そうだったので、そのまま使うことにしました。

`NewLogView` に `MainWindow` のインスタンスを渡すと、
自動で一番下に追加されたので、これでいいとおもってそのまま使いました。

`log.Fatal` はプログラムが終了してしまって意味がないので、
`log.Println` や `log.Printf` だけ使いました。

## 外部コマンド実行

[Goで外部コマンドを実行して出力をリアルタイム表示するサンプル](http://qiita.com/hnakamur/items/9701f40c1fec83b1cd1f)などを参考にして、
`StdoutPipe` と `StderrPipe` を使いました。

https://golang.org/pkg/os/exec/#Cmd.StdoutPipe や https://golang.org/pkg/os/exec/#Cmd.StderrPipe には `Wait` を呼ぶ前に
全部 `read` しろと書いてあるように思うのですが、
この Qiita の記事のやり方でそれが保証されているかどうかわからなかったのと、
読み込みの goroutine がちゃんと終了するのかどうかがわからなかったのですが、
間違っていても、多少メッセージが抜けたり、リソースがリークするだけで
機能自体に大きな問題はないと判断して、この方法で行くことにしました。

## コマンドの出力の文字コード変換

コマンドラインの方は UTF-8 のままで大丈夫なのに、
`netsh` の出力は CP932 のようだったので、
[GolangでShift_JIS(Windows31J)のファイルを読み込み - 来世から頑張る！！](http://kazzna.hatenablog.com/entry/2016/02/05/102827)などを参考にして、
`golang.org/x/text/encoding/japanese` を使って

    decoder := japanese.ShiftJIS.NewDecoder()
    scanner := bufio.NewScanner(decoder.Reader(r))

のように読み込み側だけ変換を挟むことで解決しました。

後で呼び出すコマンドラインに指定するインターフェイス名の方は、この変換後の UTF-8 の文字列のままで良いので、
非対称なのが気になります。

## 行毎の処理

[Go でファイルを1行ずつ読み込む（csv ファイルも）](http://qiita.com/ikawaha/items/28186d965780fab5533d)などを参考にして、
`bufio` の `Scanner` というのを使ってみました。

## 外部コマンド実行時のコマンドプロンプト非表示

そのまま実行してしまうと、
`netsh` 起動時にコマンドプロンプトが出てしまうので、
[【プログラミング】非表示にして起動する方法を模索:るなおーびっと！ - ブロマガ](http://ch.nicovideo.jp/lunaorbit/blomaga/ar1064613)などを参考にして、

    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

を指定すれば良いとわかり、
https://golang.org/src/syscall/exec_windows.go
を見ると
StartProcess の attr の Sys の HideWindow を true にすれば良さそうとわかりました。

最初、`cmd.SysProcAttr.HideWindow = true` としてみたら落ちてしまってうまくいかなかったので、
もう少し調べてみたところ、

    cmd.SysProcAttr = &syscall.SysProcAttr{HideWindow :true}

でうまくいきました。

## 1行入力

WALK は `TextEdit` が複数行入力だったので、1行入力はどうするのかと思って調べて見たところ、
`git grep '"EDIT"'` で探して見ると `"EDIT"` というウィンドウクラス名を使っているのは
`TextEdit` の他に `LineEdit` があるとわかったので、
`LineEdit` を試して見たところ、
1行入力でした。

## コンボボックス

IP アドレスの入力は全て入力しないといけない `LineEdit` よりも、
ある程度のプリセットが入っているコンボボックスの方が良いと思って
さらに調べて見ると、
`ComboBox` というのがあったので使って見ましたが、
HTML の `select` 要素と同じように選択できるだけで
入力できるコンボボックスになっていませんでした。

`git grep "COMBOBOX"` で探しても他にはなかったので、
さらに調べて見ると
`declarative/combobox.go` で `Editable` によって `NewComboBox` と `NewDropDownBox` を呼び分けていて、
デフォルトは `false` で `NewDropDownBox` になっていました。

そこで、`Editable: true` を追加すると、
望み通りの `ComboBox` になりました。

## 出力処理のクロージャ

ruby などに慣れていると、用途によって違う一番内側の処理はクロージャで渡したくなるので、
以下のように `func` を渡すようにしました。

最終的にコマンド実行周りは以下のようになりました。

標準エラー出力も処理していますが、
使っている範囲の `netsh` の呼び出しでは標準エラー出力には何も出てこないようです。

```
func processLinesShiftJIS(lineProcessor func(string), r io.Reader) {
        decoder := japanese.ShiftJIS.NewDecoder()
        scanner := bufio.NewScanner(decoder.Reader(r))
        for scanner.Scan() {
                line := scanner.Text()
                lineProcessor(line)
        }
}

func outputStdout(line string) {
        if line != "" {
                log.Println("o:", line)
        }
}

func outputStderr(line string) {
        if line != "" {
                log.Println("e:", line)
        }
}

func runCommand(stdoutHandler, stderrHandler func(string), name string, arg ...string) (err error) {
        cmd := exec.Command(name, arg...)
        cmd.SysProcAttr = &syscall.SysProcAttr{HideWindow: true}
        stdout, err := cmd.StdoutPipe()
        if err != nil {
                log.Println("StdoutPipe:", err)
                return
        }
        stderr, err := cmd.StderrPipe()
        if err != nil {
                log.Println("StderrPipe:", err)
                return
        }

        err = cmd.Start()
        if err != nil {
                log.Println("Start:", err)
                return
        }

        go processLinesShiftJIS(stdoutHandler, stdout)
        go processLinesShiftJIS(stderrHandler, stderr)

        err = cmd.Wait()
        if err != nil {
                log.Println("Wait:", err)
                return
        }
        return nil
}
```

## 配布

実際に配布して見ると 64 ビット環境ではなかったらしく、
エラーになってしまったので、
`GOOS=windows GOARCH=386 go build -ldflags="-H windowsgui"`
で作り直した実行ファイルを渡し直して解決しました。

念のため、
`go get` も `GOARCH=386` ありでやり直してからビルドした実行ファイルを渡したのですが、
`go get` をし直さなくても
`GOOS=windows GOARCH=386 go build -ldflags="-H windowsgui" -o winipset32.exe`
だけでもエラーなくビルドできるようでした。

## github へのリリース

`git tag v0.1.0` して `git push --tags` した後、
ブラウザーで github の Releases にもあげようとしたところ、
エラーになったので何度かやり直したのですが、
Firefox だとうまくアップロードできなくて
Chrome だとうまくアップロードできました。

Referer などのチェックが入っていて制限しているとダメなのかもしれませんが、
未調査です。

Chrome に切り替える時に `*.exe` はダメそうだったので、
`*.zip` に切り替えたのですが、その辺りも関係しているのかもしれません。

zip ファイルの作成は 7z コマンドの方が圧縮率が良いので、

    7z a winipset_windows_amd64.zip winipset.exe
    7z a winipset_windows_386.zip winipset32.exe

で作成しました。

## 感想

Go 言語で Windows の GUI アプリを作って見た感想としては、
クロスコンパイルや配布がしやすかったのは予想通りで、
非常によかったです。

WALK という GUI ツールキットも部品のサイズをいちいち指定しなくても、
良い感じにしてくれるので楽でよかったです。

部品探しやコマンド実行周りはどうしても Windows 固有の知識がないと厳しそうだと思いましたが、
ツールキット固有の知識が必要になるか、
OS の薄いラッパーなので OS の知識が必要になるかの差なので、
まあ仕方がないかなと思いました。

`log.Fatal` を使ってしまった場合など、エラーの時に黙って終了してしまうので、
用途によっては使いにくいかもしれないと思いました。
