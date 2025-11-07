---
header: Notary (Notation)
layout: resources
toc: true
show_toc: 3
description: Documentation for signing Docker images with SignPath using Notary (Notation)
---

## Overview

The _[Notary project](https://notaryproject.dev/)_ (aka Notary v2) is the successor of Docker Content Trust. It is developed by the [Cloud Native Computing Foundatio](https://cncf.io/). Both Microsoft and Amazon recommend Notation for signing container images deployed in their respective Kubernetes offerings.

Notation uses standard PKI, is easy to set up and has a mature trust model. SignPath recommends Notation for signing container images in Enterprise environments.

## Signing with SignPath

Notary signatures are created using the [Notation CLI tool](https://github.com/notaryproject/notation). SignPath provides a plugin for signing. See the [SignPath Notation Plugin](/crypto-providers/notation) page for details.

_Note: The signing happens on the client side. Only the image hash is transferred to SignPath._

