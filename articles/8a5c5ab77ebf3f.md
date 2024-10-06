---
title: "Windows Powershell環境でTmuxを使うツール"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Windows Powershell環境でTmuxを使うツール

https://github.com/ymat19/PoshOnTmux

Windowsでペイン操作をTmuxでしたい方のための記事です。
タイトルは正確ではなくて、正しくは `WSLで動かすTmux上で、ホストのPowershellを扱うツール` です。

## はじめに

本記事で紹介するツールは、[こちらのSuperUserの書き込み](https://superuser.com/a/1643117) に基づいています。この方法を基にして、今日の環境で動作するように修正し、さらに作業を自動化したものです。

## 想定ユーザ

以下のような方々が対象です。

- WindowsネイティブのPowershellで作業をする必要がある
  - 歴史的経緯等でWSLで作業が完結しないプロジェクトを扱っているなど
- Windowsターミナル等のペイン操作に不満がある
  - Leaderキーを定義したい
  - Mac等の他環境とキーバインドを統一したい

## 使い方

### インストール方法

以下のコマンドを実行して、ツールをインストールします。

```powershell
powershell -c "iwr https://raw.githubusercontent.com/ymat19/PoshOnTmux/main/install.ps1 -UseBasicParsing | iex"
```

### 起動方法

インストールが完了すると、以下のコマンドで起動できます。また、Windowsターミナルなら再起動すると勝手にプロファイルが追加されます。

```powershell
wsl -d PoshOnTmux
```

設定ファイルは次の場所にあります。

```
\\wsl$\PoshOnTmux\home\tmux\.tmux.conf
```

### アンインストール方法

以下のコマンドでアンインストールできます。

```bash
wsl --unregister -d PoshOnTmux
```

## その他

### ツールが何をしているのか

このツールは、次のステップで動作します。

1. **WSL1でAlpine Linuxインスタンスを立ち上げ**、その中でtmuxを起動。
2. **tmux内でWindows側のPowerShellを起動**し、ペイン分割や操作ができるように設定。

#### なぜWSL1なのか

WSL2をWSL1の進化版としてとらえている方もいるかもしれませんが、以下のように動作原理がかなり異なります

- WSL1
WindowsのカーネルAPIでLinuxのカーネルAPIを再現
- WSL2
ハイパーバイザ式の仮想環境でLinuxカーネルを用意

WSL1は互換性が微妙なまま今日にいたり、今ではWSL2がデファクトになりましたが、本ツールのようにシンプルなことをするだけならWSL1の方がオーバーヘッドが小さく、適してます。

### 制限事項：できないこと

> **注意**: このツールは、ペイン操作に特化しており、tmuxのセッション管理や他の高度なマルチプレクサ機能にはあまり対応していません。

- **セッションの維持ができない**  
  ローカル環境で動作しているため、PCを再起動したり、ターミナルを閉じたりするとセッションは終了します。また、既存のPowerShellセッションにアタッチすると、PowerShellが入力を受け付けなくなるという問題が発生することがあります（おま環かもしれません）。
  
- **セッションのアタッチはコメントアウト中**  
  前回のセッションに自動的にアタッチする処理自体は実装していますが、現時点ではコメントアウトしています。環境によってはうまく動作するかもしれませんので、試してみてください。

