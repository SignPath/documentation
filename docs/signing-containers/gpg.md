---
header: GPG
layout: resources
toc: true
show_toc: 3
description: Container signing with GPG
---

## Overview

Standard GPG signatures can be used to sign container images and verify them using [podman](https://podman.io/). See [Red Hat's official documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/assembly_signing-container-images_building-running-and-managing-containers) on how to perform the signing.

There is no standard way of distributing GPG-based container image signatures (e.g. in an OCI-compliant registry), they have to be transferred out-of-band to the target system.

## Signing with SignPath

See the [GPG CryptoProvider documentation](/crypto-providers/gpg) for details.

_Note: The signing happens on the client side. Only the image hash is transferred to SignPath._