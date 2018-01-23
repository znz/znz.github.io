---
layout: post
title: "Homebrew-CaskでLibreOfficeの日本語版をインストール"
date: 2018-01-23 21:55:18 +0900
comments: true
categories: homebrew osx
---
Homebrew-Cask で LibreOffice が日本語訳インターフェースも含めて簡単にインストールできるようになっていました。

<!--more-->

## バージョン

- macOS Sierra 10.12.6
- Homebrew 1.5.1
- Homebrew-Cask 1.5.1

## インストール方法

macOS を日本語環境で使っていれば `brew cask install libreoffice-language-pack` で依存関係にある `libreoffice` と一緒に入ります。

https://github.com/caskroom/homebrew-cask/blob/master/doc/cask_language_reference/stanzas/language.md に書いてあるように `brew cask install libreoffice-language-pack --language=ja` で明示的に日本語を指定することもできます。

## ダウンロードが遅い場合

日本のミラーのひとつの http://www.ftp.ne.jp/ が「 **(2017/12/21)** *ftp-srv2 is down currently*, because the network equipment for ftp-srv2 has been broken from 2017/12/19. In addition, ftp is too heavy load. Please use the other sites for archive downloads. We are trying to replace the network equipment.」と書いてあるように非常に重いので、ダウンロードが非常に遅い場合があるかもしれません。
その場合は、公式サイトから手動でミラーを選択してダウンロードした `LibreOffice_5.4.4_MacOS_x86-64.dmg` を `~/Library/Caches/Homebrew/Cask/libreoffice--5.4.4.dmg` におきます。
Cache に置く時のファイル名は、ダウンロードを中断したら `~/Library/Caches/Homebrew/Cask/libreoffice--5.4.4.dmg.incomplete` があるはずなので `.incomplete` を削ったファイル名にします。
その上で `brew cask install` を再度実行すると、ダウンロード済みのファイルが使われます。

## バージョンアップ方法

[homebrewの更新はbrew upgrade --cleanupだけでよくなっている](/blog/2017-04-27-homebrew-upgrade-cleanup.html)に書いたように `brew upgrade --cleanup` などで Homebrew の更新をした上で、 `brew cask reinstall libreoffice` で LibreOffice 自体を更新して (英語に一時的に戻って)、 `brew cask reinstall libreoffice-language-pack` で再度日本語にします。
`brew cask reinstall libreoffice libreoffice-language-pack` でまとめて更新もできます。

## まとめ

以前は LibreOffice 本体は cask でインストールできても、日本語で使おうとすると別途インストールする必要がありましたが、いつの間にか homebrew-cask が多言語対応していて、 Firefox や Thunderbird は自動で日本語が、LibreOffice の場合は LibreOffice 自体の配布がわかれているので、 `libreoffice-language-pack` を別途入れる必要がありますが、コマンド1,2個で簡単にインストールできるようになっていました。
