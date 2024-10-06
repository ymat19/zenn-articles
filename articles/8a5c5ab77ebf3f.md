---
title: "Windows Powershell環境でtmuxを使うツール"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows", "tmux", "WSL"]
published: true
---

# Windows Powershell環境でtmuxを使うツールを作りました

https://github.com/ymat19/PoshOnTmux

Windows Powershell環境で、tmuxを使ったペイン操作をしたい方のための記事です。
タイトルは正確ではなく、正しくは以下です。
`WSLで動かすtmux上で、ホスト側のPowershellを扱うツール`

※  **後述しますが、セッション管理などのマルチプレクサ機能は現状扱えません。**

## はじめに

本記事で紹介するツールは、[こちらのSuperUsern内のNotTheDr01ds氏による書き込み](https://superuser.com/a/1643117) に基づいています。これを今日の環境で動作するように修正し、セットアップを自動化したものです。

## 想定ユーザ

以下のような方々が対象です。

- WindowsネイティブのPowershellで作業をする必要がある
  - WSL上のPowershellではうまく動かない伝統ののps1
  - ネイティブのdotnetでしかビルドできないC#プロジェクト
- Windows Terminal等のペイン操作に不満がある
  - Leaderキーを定義したい
  - Macなどtmuxのある他環境とキーバインドを統一したい

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
![](/images/poshOnTmuxImage.png)

`Ctrl-b %` などしてペインが出てきたら正しくインストールできてます。
![](/images/poshOnTmuxImage2.png)

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

※ 私自身、最近まで知らなかったので補足ですが、WSLはexeを叩くとホスト側で実行してくれます。

#### なぜWSL1なのか

WSL2はWSL1の進化版のような名前ですが、実際は以下のように動作原理が異なります。

- WSL1
WindowsのカーネルAPIでLinuxのシステムコールを再現
- WSL2
ハイパーバイザ式の仮想環境でLinuxカーネルを用意

WSL1は互換性に問題があり、今ではWSL2が主流になりましたが、本ツールのようにシンプルなことをするだけならWSL1でも問題なく、さらにオーバーヘッドが小さく適してます。

#### PowershellはCore版？

Core版があればそれを起動します。なければプリインストールの方を立ち上げます。

https://github.com/ymat19/PoshOnTmux/blob/4d0ec8207b71a3e13e704d870074de8a7a516f50/setup.sh#L14

### 制限事項：できないこと

- **セッションの維持ができない**  
  ターミナルを閉じるとセッションが終了します。起動時は毎回新しいセッションを作成します。
  参考にした元スクリプトでは、起動時に既存セッションに再接続するようになっていたのですが、試してみるとなぜかちょくちょく固まるので、コメントアウトしてます。
  (tmuxやLinux側ではなく、Powershell側が固まっており、原因不明)

https://github.com/ymat19/PoshOnTmux/blob/4d0ec8207b71a3e13e704d870074de8a7a516f50/setup.sh#L21

## おわりに

最後までお読みいただきありがとうございました！
こういったツールを公開するのは初めてなので、不手際等あるかもしれません。
何か問題あればご指摘いただけると嬉しいです。

## 参考

https://superuser.com/a/1643117
