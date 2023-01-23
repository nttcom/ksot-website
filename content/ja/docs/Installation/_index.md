---
title: "Installation"
linkTitle: "Installation"
weight: 2
description: >
  Kuestaのインストール手順
---

Kuestaをインストールする手順を紹介します。

## はじめに

バージョン1.24以上のKubernetesクラスターが必要です。

Kubernetesクラスターをお持ちでない場合は、kindを用いてローカルのKubernetesクラスターを作ることができます。
[kindをインストール](https://kind.sigs.k8s.io/docs/user/quick-start/) し、`kind create cluster` を実行してクラスターを作成してください。
これにより、ローカルで動作し `cluster-admin` 権限で操作出来るKubernetesクラスターが作成されます。


## 事前準備

1. [kubectl](https://kubernetes.io/docs/tasks/tools/) をインストールしてください。Kubernetesクラスタに対してコマンドを実行する際に使用します。
2[cue](https://cuelang.org/docs/install/) をインストールしてください。Kuestaのインストールスクリプトの実行に必要です。


## Kuestaのインストール


Kuestaをインストールするには、以下のコマンドを実行してください。

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta/tool/install
cue install
```

リリースバージョンを指定したい場合、もしくは別のコンテナレジストリを使用してインストールしたい場合は、 `version` もしくは `imageRegistry` タグを使用してください。

```bash
cue -t imageRegistry=<your_image_registry> -t version=<your_version> install
```

プロンプトが表示されますので、指示に従ってパラメータを入力してください。

```bash
GitHub repository for config: https://github.com/<your_org>/<your_config_repo>
GitHub repository for status: https://github.com/<your_org>/<your_status_repo>
GitHub username: <your_username>
GitHub private access token: <your_private_access_token>
Are these git repositories private? (yes|no): no
Do you need sample driver and emulator for trial?: (yes|no) yes

---
GitHub Config Repository: https://github.com/<your_org>/<your_config_repo>
GitHub Status Repository: https://github.com/<your_org>/<your_status_repo>
GitHub Username: <your_username>
GitHub Access Token: ***
Use Private Repo: true

Image Registry: ghcr.io/nttcom/kuesta
Version: latest
Deploy sample driver and emulator: true
---

Continue? (yes|no) yes
```

KuestaはGitへのプッシュやプルリクエストの作成を行うため、GitHubの [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) (PAT) が必要です。
上記で入力したPATは `kuesta-system/kuesta-secrets` という `Secret` リソースに保存され、kuestaサーバが git clone・pull・push・PullRequest作成を行うために使用されます。

また、非公開のGitレポジトリサーバに対してHTTPSで接続したい場合は、`GitRepository` を作成する際にこのPATを使うこともできます。Flux source-controllerにGitプライベートレポジトリへのアクセスを許可するために使用されます。


## Flux source-controllerを用いてGitOpsを設定する

Flux source-controllerを用いてGitOpsを行うには、IaCマニフェストを取得するソースを定義するために `GitRepository` リソースを作成する必要があります。
以下は、[getting-started](/docs/getting-started) チュートリアルで作成された `GitRepository` のサンプルです。

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: kuesta-testdata
  namespace: kuesta-getting-started
spec:
  interval: 1m0s
  url: https://github.com/hrk091/kuesta-testdata
  ref:
    branch: main
  secretRef:
    name: kuesta-testdata-secret
  timeout: 60s
```

`GitRepository`に関するより詳細な情報が知りたければ、[Fluxのドキュメント](https://fluxcd.io/flux/components/source/gitrepositories/)を参照してください。

Kuestaでは、`GitRepository`を作成するとKuestaのsource-watcherコントローラが検知して、対応する `DeviceRollout` を作成します。
`DeviceRollout` は、マルチデバイスのトランザクション管理を責務とするカスタムリソースです。
getting-started チュートリアルでは、以下の `DeviceRollout` が作成されます。

```
apiVersion: kuesta.hrk091.dev/v1alpha1
kind: DeviceRollout
metadata:
  name: kuesta-testdata
  namespace: kuesta-getting-started
spec:
  deviceConfigMap:
    oc01:
      checksum: 1524caf17ec4c133f6f275326a6c9742a0155007f8eaa28ea601ce7ac39c0566
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
    oc02:
      checksum: 0b03381195ba613e0dd20e5d2683711e9d6082f0634bee7c54014de2eadf0fc9
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
status:
  desiredDeviceConfigMap:
    oc01:
      checksum: 1524caf17ec4c133f6f275326a6c9742a0155007f8eaa28ea601ce7ac39c0566
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
    oc02:
      checksum: 0b03381195ba613e0dd20e5d2683711e9d6082f0634bee7c54014de2eadf0fc9
      gitRevision: main/30746f9beb48d237b6c0e0357708b3ed1f95cd1b
  deviceStatusMap:
    oc01: Completed
    oc02: Completed
  phase: Healthy
  status: Completed      
```

`DeviceRollout` の管理はライフサイクル全体を通じて自動化されていますので、作成や変更を行う必要はありません。


## Installing Device Driver and configuring Device Custom Resource

{{% pageinfo %}}

以降の記述は、getting-started チュートリアルで使用されているサンプルのKubernetesカスタムオペレータを使用した場合に限定した内容になります。
Kuestaを用いて制御したいターゲットデバイスと、その制御のために実装したKubernetesカスタムオペレータによって、詳細な手順は異なります。

{{% /pageinfo %}}

OpenConfig over gNMIをサポートしているデバイスに設定を投入したい場合は、[kuesta/device-operator](https://github.com/nttcom/kuesta/tree/main/device-operator) で開発されているサンプルのKubernetesカスタムオペレータを利用できます。
このサンプルのKubernetesカスタムオペレータは[基本的なOpenConfig YANGセット](https://github.com/nttcom/kuesta/tree/main/device-operator/pkg/model/yang)をサポートしているデバイスに特化していますが、[OpenConfigスタイル](https://github.com/openconfig/public/blob/master/doc/openconfig_style_guide.md)に準拠しているYANGであれば任意に追加して拡張することが可能です。

このKubernetesカスタムオペレータは `make` コマンドもしくは `kubectl/kustomize` を直接実行することでインストールできますが、
Kuestaのインストールスクリプトを使用するのが一番簡単です。Kuestaのインストールスクリプトを実行すると `getting-started` という名前でkustomizeのオーバレイディレクトリが作成されますので、これをテンプレートとして自由にカスタムできます。

`getting-started` オーバレイディレクトリを用いてKubernetesカスタムオペレータをインストールするには、以下のコマンドを実行します:

```bash
export IMG=ghcr.io/nttcom/kuesta/device-operator:latest
export KUSTOMIZE_ROOT=overlays/getting-started
make deploy
```

Kuestaからデバイスに対して設定投入するためには、Kubernetesカスタムオペレータに加えて、各デバイスごとにカスタムリソースを作成する必要があります。
getting-startedチュートリアルでは、gNMIエミュレータ `oc01` に対して以下のような `OcDemo` カスタムリソースを作成します。

```yaml
apiVersion: kuesta.hrk091.dev/v1alpha1
kind: OcDemo
metadata:
  name: oc01
  namespace: kuesta-getting-started
spec:
  address: gnmi-fake-oc01.kuesta-getting-started
  port: 9339
  rolloutRef: kuesta-testdata
  tls:
    secretName: oc01-cert
    skipVerify: true
```

`OcDemo` リソースでは、設定投入を行うターゲットデバイスの接続情報、アドレス、ポート、ユーザ名とパスワード、TLS関連設定、などを定義します。
`.spec.rolloutRef` フィールドは、Flux source-controllerのGitOpsによって提供されるコンフィグの更新通知を、どの `DeviceRollout` から取得するかを指定するフィールドであり、必須項目です。
`OcDemo` リソースの `.spec` についてより詳細な情報が知りたければ、 `kubectl explain ocdemo.spec` を実行して確認してください。
