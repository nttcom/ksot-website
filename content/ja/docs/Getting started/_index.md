---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 1
description: >
  Kuestaを動かしてみよう
---

このチュートリアルでは、以下のステップでKuestaを構築し、動作確認を行います。

1. ローカルのKubernetesクラスターを作成する
2. Kuestaをインストールする
3. 装置エミュレータを立ち上げる
4. 新しいServiceコンフィグを作成して、GitOpsを介して装置エミュレータに設定を投入する


## 事前準備 

1. [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) をインストールしてください。ローカルのKubernetesクラスタの構築に使用します。
2. [kubectl](https://kubernetes.io/docs/tasks/tools/) をインストールしてください。Kubernetesクラスタに対してコマンドを実行する際に使用します。
3. [cue](https://cuelang.org/docs/install/) をインストールしてください。Kuestaのインストールスクリプトの実行に必要です。
4. [gnmic](https://gnmic.kmrd.dev/install/) をインストールしてください。Kuestaサーバに対してgNMIリクエストを行うために必要です。
5. [golang](https://go.dev/doc/install) をインストールしてください。カスタムコントローラーのツール類を使用するために必要です。

## ローカルKubernetesクラスタの立ち上げ

以下のコマンドを実行して、ローカルのKubernetesクラスタを立ち上げます。

```bash
kind create cluster
```

実行が完了したら、以下のコマンドを実行して、Kubernetesクラスタが正常に作成されたことを確認します。

```bash
kubectl cluster-info
```

以下のような出力が表示されたら、ローカルクラスタは正常に動作しています。

```bash
Kubernetes control plane is running at https://127.0.0.1:52096
CoreDNS is running at https://127.0.0.1:52096/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


## 検証用のGithubレポジトリの作成

KuestaのGitOpsを用いてネットワーク装置に設定を行うには、2つのGitHubレポジトリを作成する必要があります。
1つ目はGitOpsの起点となるコンフィグ管理レポジトリで、2つ目はネットワーク装置の実際のコンフィグの変更履歴を管理するレポジトリです。
[Kuesta Example](https://github.com/nttcom/kuesta-example) レポジトリをテンプレートとして使用することで、これらのレポジトリを簡単に作成できます。

以下を2回実行して、2つのGitHubレポジトリを作成してください。

1. [Kuesta Example](https://github.com/nttcom/kuesta-example) レポジトリの **Use this template** ボタンをクリックしてください。

2. レポジトリ名を入力して、 **Create repository from template** ボタンをクリックしてください。レポジトリの公開・非公開については、好きな方を選択してください。


## Kuestaと検証用サンプルリソースのインストール

Kuestaと、検証用のドライバオペレータや装置エミュレータなどのサンプルリソースをインストールするには、以下のコマンドを実行してください。

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta/tool/install
cue install
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
ここで入力されたPATは、ローカルのKubernetesクラスタ内のSecretリソースにのみ保存されます。ローカルのKubernetesクラスタを削除することで、どこにもデータを残さずに完全に削除できます。

インストールスクリプトの実行が完了したら、 `kubectl` コマンドを実行することで何がインストールされたかを確認できます。

1: Custom Resource Definitions(CRDs):
```bash
kubectl get customresourcedefinitions.apiextensions.k8s.io
````

```bash
NAME                                        CREATED AT
buckets.source.toolkit.fluxcd.io            2023-01-10T06:31:20Z
certificaterequests.cert-manager.io         2023-01-10T06:31:19Z
certificates.cert-manager.io                2023-01-10T06:31:19Z
challenges.acme.cert-manager.io             2023-01-10T06:31:19Z
clusterissuers.cert-manager.io              2023-01-10T06:31:19Z
devicerollouts.kuesta.hrk091.dev            2023-01-10T06:43:30Z
gitrepositories.source.toolkit.fluxcd.io    2023-01-10T06:31:20Z
helmcharts.source.toolkit.fluxcd.io         2023-01-10T06:31:20Z
helmrepositories.source.toolkit.fluxcd.io   2023-01-10T06:31:20Z
issuers.cert-manager.io                     2023-01-10T06:31:19Z
ocdemoes.kuesta.hrk091.dev                  2023-01-10T06:43:36Z
ocirepositories.source.toolkit.fluxcd.io    2023-01-10T06:31:20Z
orders.acme.cert-manager.io                 2023-01-10T06:31:19Z
```

`kuesta.hrk091.dev`、`source.toolkit.fluxcd.io`、`cert-manager.io` に属するCRDが追加されていることが確認できます。


2: Pods:
```bash
kubectl get pods -A | grep -v 'kube-system'
```

```bash
NAMESPACE                NAME                                                  READY   STATUS    RESTARTS   AGE
cert-manager             cert-manager-7475574-t7qlm                            1/1     Running   0          12m
cert-manager             cert-manager-cainjector-d5dc6cd7f-wbh6r               1/1     Running   0          12m
cert-manager             cert-manager-webhook-6868bd8b7-kc85s                  1/1     Running   0          12m
device-operator-system   device-operator-controller-manager-5d8648469b-57x4l   2/2     Running   0          11s
flux-system              source-controller-7ff779586b-wsgf7                    1/1     Running   0          12m
kuesta-getting-started   gnmi-fake-oc01-79d4d679b4-tvm78                       1/1     Running   0          10s
kuesta-getting-started   gnmi-fake-oc02-69d9cc664d-56bsz                       1/1     Running   0          10s
kuesta-getting-started   subscriber-oc01                                       1/1     Running   0          9s
kuesta-getting-started   subscriber-oc02                                       1/1     Running   0          9s
kuesta-system            kuesta-aggregator-7586999c47-jj7m4                    1/1     Running   0          27s
kuesta-system            kuesta-server-85fc4f8646-kq72c                        1/1     Running   0          27s
local-path-storage       local-path-provisioner-9cd9bd544-lk9w9                1/1     Running   0          16m
provisioner-system       provisioner-controller-manager-7675d487c8-w6f7x       1/2     Running   0          17s
```

全てのPodがRunningのステータスになっていたら、準備完了です。

以下の各namespaceに、Kuestaのコア機能として必要なサービスがデプロイされています。

- kuesta-system
  - Kuestaのコア機能やAPIを提供するサーバ
- provisioner-system
  - KuestaのKubernetesカスタムオペレータ。GitOpsやトランザクション管理に使用
- flux-system
  - FluxCDのカスタムコントローラ
- cert-manager
  - プライベートCAの認証局、およびmTLS用の証明書発行

また、以下のnamespaceには、本チュートリアル用のリソースがデプロイされています。

- device-operator-system
  - チュートリアルで使用する装置エミュレータに対し設定投入を行うためのドライバとして振る舞うKubernetesカスタムオペレータ
- kuesta-getting-started
  - チュートリアルで使用する装置エミュレータやエミュレータに設定を行うためのKubernetesカスタムリソースなど

以下の2つは、OpenConfig over gNMIで設定が可能な装置エミュレータです。本チュートリアルでは、 `oc01` と `oc02` の2つのエミュレータがデプロイされています。

```bash
kuesta-getting-started   gnmi-fake-oc01-79d4d679b4-tvm78                       1/1     Running   0          10s
kuesta-getting-started   gnmi-fake-oc02-69d9cc664d-56bsz                       1/1     Running   0          10s
```

ポートフォワーディングして `gnmic` からリクエストを行うことで設定されているコンフィグを確認できますので、必要に応じて実施してください。
以下のコマンドで、装置エミュレータ内のコンフィグの確認ができます。

```bash
# oc01のコンフィグを取得
kubectl -n kuesta-getting-started port-forward svc/gnmi-fake-oc01 9339:9339
gnmic -a :9339 -u admin -p admin get --path "" --encoding JSON --skip-verify
```


## Serviceコンフィグを作成し、GitOpsを走らせる

**Service** は、ネットワークコンフィグを高レベルに抽象化したモデルです。Serviceを作成/変更することで、関連する複数のネットワーク装置にまとめて設定を投入することができます。

本チュートリアル用のサンプルレポジトリの中には、 `oc-circuit` という名のServiceが設定されていますので、これを用いて動作確認を行います。
ServiceはCUEで記述されます。例として、 `oc-circuit` Serviceの定義を以下に示します。

```cue
package oc_circuit

import (
	ocdemo "github.com/hrk091/kuesta-testdata/pkg/ocdemo"
)

#Input: {
	// kuesta:"key=1"
	vlanID: uint32
	endpoints: [...{
		device: string
		port:   uint32
	}]
}

#Template: {
	input: #Input

	output: devices: {
		for _, ep in input.endpoints {
			"\(ep.device)": config: {
				ocdemo.#Device

				let _portName = "Ethernet\(ep.port)"
				Interface: "\(_portName)": SubInterface: "\(input.vlanID)": {
					Ifindex:     ep.port
					Index:       input.vlanID
					Name:        "\(_portName).\(input.vlanID)"
					AdminStatus: 1
					OperStatus:  1
				}

				Vlan: "\(input.vlanID)": {
					VlanId: input.vlanID
					Name:   "VLAN\(input.vlanID)"
					Status: ocdemo.#Vlan_Status_ACTIVE
					Tpid:   ocdemo.#OpenconfigVlanTypes_TPID_TYPES_UNSET
				}
			}
		}
	}
}
```

`oc-circuit` Serviceは、`#Template` フィールドに定義された内容に従い、受け取った `#Template.input`（Serviceコンフィグ）を `#Template.output` （複数のネットワーク装置のコンフィグ）に変換します。
`#Input` の定義されているServiceのインターフェーススキーマを見ると、`oc-circuit` Serviceを使用するためには、 VLAN ID と複数のネットワーク装置のポートが必要なことがわかります。
これらを指定して `oc-circuit` Serviceを作成すると、 `#Template` のマッピング定義に従い、VLAN定義と指定したポートのVLANサブインターフェースが作成されます。

以下の手順にしたがって、`oc-circuit` Serviceを作成してください。

1: `oc-circuit` Serviceの作成をリクエストするために、以下のJSONファイルを `oc-circuit-vlan100.json` という名前で保存してください。
このServiceでは、oc01とoc02の両エミュレータに対して、ポート番号1のインターフェイスにVLAN ID=100のVLANサブインターフェースを作成することをリクエストしています。

```json
{
    "vlanID": 100,
    "endpoints": [
        {
            "device": "oc01",
            "port": 1
        },
        {
            "device": "oc02",
            "port": 1
        }
    ]
}
```

2: ローカルのKubernetesクラスタで動作しているKuestaサーバのPodに対してポートフォワーディングしてください。

```bash
kubectl -n kuesta-system port-forward svc/kuesta-server 9339:9339
```

3: `gnmic` を用いて、Kuestaサーバに対してgNMI SetRequestを送ってください。

```bash
gnmic -a :9339 -u dummy -p dummy set --replace-path "/services/service[kind=oc_circuit][vlanID=100]" --encoding JSON --insecure --replace-file oc-circuit-vlan100.json
```

4: GitHubのWebコンソールを用いて、本チュートリアル向けに作成したコンフィグ用のGitHubレポジトリのプルリクエスト(PullRequest: PR)を確認してください。
[PR一覧](https://github.com/<your_org>/<your_config_repo>/pulls) にアクセスすると、 `[kuesta] Automated PR` というタイトルのPRが確認できます。
PRのコメントを見ると、どのServiceとどのネットワーク装置が変更されたのかがわかりますし、 `Files changed` タブを確認すると詳細な変更点が確認できます。

5: PRブランチをマージする前に、ローカルのKubernetesクラスターで `DeviceRollout` リソースをモニターしてください。 `DeviceRollout` リソースは、Gitレポジトリからプルしたコンフィグを各ネットワーク装置に設定する際の状態管理を行っています。以下のコマンドを実行してください。 

```bash
kubectl -n kuesta-getting-started get devicerollouts.kuesta.hrk091.dev -w
```

`DeviceRollout` リソースが一つ表示され、その `STATUS` が `Completed` になっていれば、GitOpsによるデリバリプロセスを流す準備は完了です。

```bash
NAME              PHASE     STATUS
kuesta-testdata   Healthy   Completed
```


6: GitHubのWebコンソール上で、PRをマージしてください。その後、 `DeviceRollout` をモニターしているターミナル画面に移動してください。
1分以内に、 `DeviceRollout` の `STATUS` が `Running` に変わり、その後 `Completed` に変わることを確認してください。

```bash
NAME              PHASE     STATUS
kuesta-testdata   Healthy   Completed
kuesta-testdata   Healthy   Completed
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Running
kuesta-testdata   Healthy   Completed
```

7: 装置コンフィグが本当に更新されたのかを確認します。 `gnmic` を用いて、gNMI GetRequestをKuestaサーバに対して送り、装置コンフィグ全体を取得してください。

```bash
gnmic -a :9339 -u admin -p admin get --path "/devices/device[name=oc01]" --path "/devices/device[name=oc02]" --encoding JSON --insecure
```

以下のようなレスポンスが表示されます。

```json
[
  {
    "source": ":9339",
    "timestamp": 1673360105000000000,
    "time": "2023-01-10T23:15:05+09:00",
    "updates": [
      {
        "Path": "devices/device[name=oc01]",
        "values": {
          "devices/device": {
            "Interface": {
              "Ethernet1": {
                "AdminStatus": 1,
                "Description": "awesome interface!!",
                "Enabled": false,
                "Mtu": 9000,
                "Name": "Ethernet1",
                "OperStatus": 1,
                "Subinterface": {
                  "100": {
                    "AdminStatus": 1,
                    "Ifindex": 1,
                    "Index": 100,
                    "Name": "Ethernet1.100",
                    "OperStatus": 1
                  },
                  ...
                },
                ...
              },
              ...
            },
            "Vlan": {
              "100": {
                "Member": [],
                "Name": "VLAN100",
                "Status": 1,
                "Tpid": 0,
                "VlanId": 100
              },
              ...
```

レスポンスを確認すると、oc01とoc02の両方の装置エミュレータに対して、VLAN ID=100のVLAN定義とVLANサブインターフェースが設定されていることが分かります。
また、これらの装置コンフィグの変更内容はチュートリアル向けに作成したステータス用のGitレポジトリに対してコミット・プッシュされています。ステータス用のGitレポジトリのメインブランチのHEADを見ると、変更が保存されていることが確認できます。


## ローカルKubernetesクラスタの削除

このチュートリアルで作成したローカルのKubernetesクラスタを削除するには、以下のコマンドを実行してください。

```bash
kind delete cluster
```

プライベートレポジトリのアクセス用に作成したPATは、不要であれば忘れずに削除しましょう。
