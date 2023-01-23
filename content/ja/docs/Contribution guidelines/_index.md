---
title: "Contribution Guidelines"
linkTitle: "Contribution Guidelines"
weight: 10
description: How to contribute to Kuesta
---

Kuesta is an open source project and we love getting patches and contributions to make Kuesta and its docs even better.


## Contributing to Kuesta


### Creating PullRequest

All submissions, including submissions by project members, require review. We
use GitHub PullRequest for this purpose. Consult
[GitHub Help](https://help.github.com/articles/about-pull-requests/) for more
information on using pull requests.


### Creating issues

If you have found something that isn't working the way you'd expect, but you're not sure how to fix it yourself, please create an [issue](https://github.com/nttcom/kuesta/issues).


## How to run the test suites

To run the whole test suites of all kuesta components:

```bash
make test-all
```

To run the specified component test suites only, run the following at the top of the component dir:

```bash
make test
```

For lint, install [golangci-lint](https://golangci-lint.run/usage/install/#local-installation) in advance, then run at the top of kuesta repo:

```bash
make lint-all
```

If you want to run Kuesta locally, see ["Installtaion"](/docs/installation) page.
