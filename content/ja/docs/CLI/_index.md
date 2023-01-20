---
title: "CLI"
linkTitle: "CLI"
weight: 5
description: >
  kuesta CLIの使い方
---

Kuestaでは、コンフィグIaCを更新する方式として、kuesta-serverを使ってAPI経由で更新する方式と `kuesta` CLIを用いる方式をサポートしています。
`kuesta` CLIは、 Kuestaを用いつつも手動でコンフィグレポジトリの管理をしたい場合や、GitHub ActionsやGitHub Appsなどと連携してCIを構成する場合に役立ちます。


## インストール

Kuestaのtarファイルを、 [kuesta Releases page](https://github.com/nttcom/kuesta/releases) からダウンロードしてください。
ダウンロードできたら展開して、PATHが通っているディレクトリに実行バイナリを配置してください。

```bash
# Replace YOUR-DOWNLOADED-FILE with the file path of your own.
sudo tar xvzf YOUR-DOWNLOADED-FILE -C /usr/local/bin/ kuesta
```

### ソースコードからビルド

`kuesta` CLIをソースコードからビルドしたい場合は、Go1.18以上を用いた開発環境をセットアップしたのち、以下のコマンドを実行してください。

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta
go build -o kuesta main.go
```

その後、ビルドされたバイナリをPATHが通っているディレクトリに配置してください。


## 使い方

kuesta-serverを使用せずに、`kuesta` CLIを用いてコンフィグIaCを更新するには、以下のステップを実行してください。

1: ネットワークコンフィグを管理しているGitHubマニフェストレポジトリを、クローンしてください。

2: Serviceコンフィグを更新してください。

3: 更新したServiceに `git add` を実行し、インデックスに追加してください。

4: Serviceコンフィグからネットワーク装置のコンフィグに変換するために、 `kuesta service apply` を以下の通り実行してください。

```bash
kuesta service apply -p=<path_to_config_repo_root>
```

Serviceコンフィグからネットワーク装置へのコンフィグ変換は、インデックスに追加されているServiceに限定して実行されます。
インデックスに追加されていないServiceの変更は全てスキップされます。


5: GitHubマニフェストレポジトリ上でプルリクエストを作成するために、 `kuesta git commit` を以下の通り実行してください。
もしgitプロバイダへの認証が必要な場合は、コマンドラインオプション `--git-token` か `KUESTA_GIT_TOKEN` を用いて、認証済のアクセストークンを指定してください。


```bash
kuesta git commit -p <path_to_config_repo_root> -r <config_repo_url> 
```

その他の利用可能なコマンドや、コマンドラインオプションに関するより詳細な情報が知りたい場合は、 `kuesta help` を実行してください。
