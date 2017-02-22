---
layout: post
title: "Go言語のsync.WaitGroupで出力待ち"
date: 2017-02-22 22:22:22 +0900
comments: true
categories: golang
---
[みんなのGo言語【現場で使える実践テクニック】](http://amzn.to/2l3wZ5L)という本に `sync.WaitGroup` というのを知ったので使ってみました。

<!--more-->

## 環境

- `go version go1.8 darwin/amd64`

## `sync.WaitGroup` の追加

呼び出し側は以下のように追加しました。

```
	var wg sync.WaitGroup
	wg.Add(2)
	go processLinesShiftJIS(stdoutHandler, stdout, &wg)
	go processLinesShiftJIS(stderrHandler, stderr, &wg)
	wg.Wait()
```

呼び出される側では、以下のように最後に `wg.Done()` を呼ぶようにしました。
記事を書いていて気づいたのですが、途中で抜ける可能性を考えて、 `defer` を使った方が良さそうです。

```
func processLinesShiftJIS(lineProcessor func(string), r io.Reader, wg *sync.WaitGroup) {
	// ...処理...
	wg.Done()
}
```

## 動かなかったコード

最初は `func processLinesShiftJIS(lineProcessor func(string), r io.Reader, wg sync.WaitGroup)` で受け取って `go processLinesShiftJIS(stdoutHandler, stdout, wg)` で呼び出していました。

これだと、`WaitGroup` のコピーで `Done()` が呼ばれてしまって、元の `WaitGroup` のカウントが減らずにうまく動きませんでした。

本に書いてあった例やネットで見つけた例は同じスコープで使っているものばかりだったので
わからなかったのですが、
ちゃんとポインタを渡す必要があるようです。

## 関連コード全体

全体としては関係する部分の [winipset.go](https://github.com/znz/winipset/blob/cb4a80ebdaf3c1492bb81f0654424205cf8aa87e/winipset.go) を以下のように変更しました。

```
func processLinesShiftJIS(lineProcessor func(string), r io.Reader, wg *sync.WaitGroup) {
	decoder := japanese.ShiftJIS.NewDecoder()
	scanner := bufio.NewScanner(decoder.Reader(r))
	for scanner.Scan() {
		line := scanner.Text()
		lineProcessor(line)
	}
	wg.Done()
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

	var wg sync.WaitGroup
	wg.Add(2)
	go processLinesShiftJIS(stdoutHandler, stdout, &wg)
	go processLinesShiftJIS(stderrHandler, stderr, &wg)
	wg.Wait()

	err = cmd.Wait()
	if err != nil {
		log.Println("Wait:", err)
		return
	}
	return nil
}
```

## 同期方法の確認

この同期方法で問題が起きなかった `cmd.Wait()` を呼ばないと `wg.Done()` が呼ばれないということはなかった、という点と、
出力をちゃんと読んだ後で `cmd.Wait()` を呼べるようになったという点で、この方法で大丈夫だということがわかりました。

## まとめ

`sync.WaitGroup` を他の関数へ受け渡す時に失敗してしまうと同期に失敗してしまうことがありましたが、
簡単に同期が取れて、使いやすい仕組みだと思いました。
