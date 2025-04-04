---
title: "home.nixをFlakes移行する【home-manager】"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nix", "dotfiles"]
published: true
---

# home.nixをFlakesに移行する

vim-jpのslackで有識者の書き込みを見たりしながら、home.nixをFlakes移行しました。  
コミュニティに圧倒的感謝。

「Flakesとは！何が嬉しいのか！」みたいな話はしないです。なぜなら私自身まだよく分かってないため。  
とりあえず移行に成功したので手順を残しておきます。

## とりあえずflake.nixを使えるようにする

home.nixがあるディレクトリで以下を実行

```bash
home-manager init .
```

するとこんなのが出てくる。

```nix: flake.nix

{
  description = "Home Manager configuration of hoge-user";

  inputs = {
    # Specify the source of Home Manager and Nixpkgs.
    nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
    home-manager = {
      url = "github:nix-community/home-manager";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { nixpkgs, home-manager, ... }:
    let
      system = "aarch64-darwin";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      homeConfigurations."hoge-user" = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;

        # Specify your home configuration modules here, for example,
        # the path to your home.nix.
        modules = [ ./home.nix ];

        # Optionally use extraSpecialArgs
        # to pass through arguments to home.nix
      };
    };
}
```

`hoge-user` というのはユーザ名が入るはずで、このflakeを用いる時に指定しないといけない。  
これは多分プロファイル的な概念で、複数の構成を用意しておける？[^1]

そしたら以下でswitchできます

```bash
home-manager switch --flake .#hoge-user
```

一般的な構成ならそのまま成功するはず、お疲れ様でした！  
(gitのsubmoduleがあるとうまくいかないです。私は諦めてレポジトリ統合しました。)

私はこれでneovimのバージョンが`0.10.04`→`0.11.0`になりました。神  
パッケージレポジトリが中央集権的なchannelから、分散管理的なFlakesに移行することで新しいパッケージが使いやすくなるという理解[^2]

## 良い感じにしてみる

### はじめに

これから書く２点のいずれかでも実施するとflakeがpureじゃなくなるので、実行コマンドに`--impure`が必要になります。

```bash
home-manager switch --flake .#hoge-user --impure
```

### 1. マルチアーキテクチャに使えるようにする

initで生成されるflake.nixは実行環境のアーキテクチャで固定されてます
どこでも使えるようにしたいなら↓

```diff nix: flake.nix
   outputs = { nixpkgs, home-manager, ... }:
     let
-      system = "aarch64-darwin";
+      system = builtins.currentSystem;
       pkgs = nixpkgs.legacyPackages.${system};
     in {
```

### 2. 環境情報をhome.nixに注入する

home.nixはpureなまま、flakeで`$USER`や`$HOME`を読めるようになるので素敵（なはず）

```diff nix: flake.nix
   outputs = { nixpkgs, home-manager, ... }:
     let
       system = builtins.currentSystem;
       pkgs = nixpkgs.legacyPackages.${system};
+      envUsername = builtins.getEnv "USER";
+      envHomeDir  = builtins.getEnv "HOME";
     in {
       homeConfigurations."hoge-user" = home-manager.lib.homeManagerConfiguration {
         inherit pkgs;
@@ -25,6 +27,10 @@

         # Optionally use extraSpecialArgs
         # to pass through arguments to home.nix
+        extraSpecialArgs = {
+          username = envUsername;
+          homeDirectory = envHomeDir;
+        };
       };
     };
 }
```

`home.nix`はinput使うだけ

```nix: home.nix

{ config, pkgs, username, homeDirectory, ... }:

let
   // 略
in
{
  home.username = username;
  home.homeDirectory = homeDirectory;
```

## CI (依存関係の自動アップデート)

flakeにはlockファイルがあるので、CIで更新したいところですが残念ながらdependabotは非対応。

しかし安心してください、先人による更新用のworkflowが以下にあります。神

https://github.com/orhun/binsider/blob/d3b90ad5a080f49e109e04be1956eeccd296d1df/.github/workflows/flake-update.yml

[^1]: よく分かってない

[^2]: よく分かってない
