---
title: "CLI"
linkTitle: "CLI"
weight: 5
description: >
  Instructions of Command-Line Interface.
---

Kuesta supports IaC configuration update via both gNMI and `kuesta` CLI.
`kuesta` CLI helps you when you would like to manage your configuration repository manually using `kuesta`, or integrate with CI like GitHub Actions or GitHub Apps.


## Installation

Download `kuesta` CLI as a tarball from the [kuesta Releases page](https://github.com/nttcom/kuesta/releases).
After downloading the tarball, extract it to your PATH:

```bash
# Replace YOUR-DOWNLOADED-FILE with the file path of your own.
sudo tar xvzf YOUR-DOWNLOADED-FILE -C /usr/local/bin/ kuesta
```

### Build from source

If you would like to build kuesta from the source, set up your Go environment using Go 1.18 or higher, and run the following:

```bash
git clone https://github.com/nttcom/kuesta.git
cd kuesta
go build -o kuesta main.go
```

Then move built binary to your PATH. 


## Usage

You can perform an IaC configuration update without using kuesta-server by the following steps:

1: Clone the GitHub Repository you want to use as the configuration manifest source.

2: Update service configs.

3: Add these updated services to git Index.

4: Run `kuesta service apply` to perform data mapping from the service config to the device config as follows:

```bash
kuesta service apply -p=<path_to_config_repo_root>
```

Data mapping from the service config to the device config will be performed only for git-staged services, all unstaged changes will be skipped. 

5: Run `kuesta git commit` to create git PullRequest on your configuration manifest repository. If authentication to git provider is required,
use `--git-token` command flag or `KUESTA_GIT_TOKEN` environment value to provide GitHub private access token.

```bash
kuesta git commit -p <path_to_config_repo_root> -r <config_repo_url> 
```

For more information about available commands and flags, run `kuesta help`.
