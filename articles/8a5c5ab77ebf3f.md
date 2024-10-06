---
title: "Windows Powershell環境でtmuxを使うツール"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "tmux"]
published: false
---

# Windows Powershell環境でtmuxを使うツール

https://github.com/ymat19/PoshOnTmux

Windows Powershell環境で、ペイン操作をtmuxでしたい方のための記事です。
タイトルは正確ではなくて、正しくは `WSLで動かすtmux上で、ホスト側のPowershellを扱うツール` です。

※  後述しますが、セッション管理などのマルチプレクサ機能は現状扱えません。

## はじめに

本記事で紹介するツールは、[こちらのSuperUserの書き込み](https://superuser.com/a/1643117) に基づいています。この方法を基にして、今日の環境で動作するように修正し、さらに作業を自動化したものです。

## 想定ユーザ

以下のような方々が対象です。

- WindowsネイティブのPowershellで作業をする必要がある
  - WSL上のPowershellではうまく動かない秘伝のps1
  - ネイティブのdotnetでしかビルドできないC#プロジェクト
- Windowsターミナル等のペイン操作に不満がある
  - Leaderキーを定義したい
  - Mac等の他環境とキーバインドを統一したい

## 使い方

### インストール方法

以下のコマンドを実行して、ツールをインストールします。

```powershell
powershell -ExecutionPolicy Bypass -c "iwr https://raw.githubusercontent.com/ymat19/PoshOnTmux/main/install.ps1 -UseBasicParsing | iex"
```

### 起動方法

インストールが完了すると、以下のコマンドで起動できます。

```powershell
wsl -d PoshOnTmux
```

また、Windowsターミナルなら再起動すると勝手にプロファイルが追加されます。
![](/images//poshOnTmuxImage.png)

### 設定ファイル

設定ファイルは次の場所にあります。

```
\\wsl$\PoshOnTmux\home\tmux\.tmux.conf
```

### アンインストール方法

以下のコマンドでアンインストールできます。

```bash
wsl --unregister -d PoshOnTmux
```

## 補足

### ツールが何をしているのか

このツールは、次のステップで動作します。

1. **WSL1でAlpine Linuxインスタンスを立ち上げる**
2. **tmuxでWindows側のPowerShellを起動** 

※ 私自身、最近まで知らなかったので補足ですが、WSLはexeを叩くとホスト側で起動してくれます。

#### なぜWSL1なのか

WSL2はWSL1の進化版のような名前ですが、実際は以下のように動作原理が異なります。

- WSL1
WindowsのカーネルAPIでLinuxのシステムコールを再現
- WSL2
ハイパーバイザ式の仮想環境でLinuxカーネルを用意

WSL1は互換性に問題があり、今ではWSL2がデファクトになりましたが、本ツールのようにシンプルなことをするだけならWSL1の方がオーバーヘッドが小さく、適してます。

#### PowershellはCore版？

Core版があればそっちを起動します。なければプリインストールの方を立ち上げます。
https://github.com/ymat19/PoshOnTmux/blob/4d0ec8207b71a3e13e704d870074de8a7a516f50/setup.sh#L14

### 制限事項：できないこと

- **セッションの維持ができない**  
  ターミナルを閉じるとセッションが終了します。起動時は毎回新しいセッションを作るようにしてます。
  参考にした元スクリプトでは、起動時に既存セッションに再接続するようになっていたのですが、試してみるとなぜかちょくちょく固まるので、コメントアウトしてます。
  (tmuxやLinux側ではなく、Powershell側が固まっており、原因不明)
  https://github.com/ymat19/PoshOnTmux/blob/4d0ec8207b71a3e13e704d870074de8a7a516f50/setup.sh#L21

## 参考

https://superuser.com/a/1643117
