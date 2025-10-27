header: Notation Plugin
layout: resources
toc: true
show_toc: 3
description: SignPath Notation Plugin

## General instructions

[notation] is a command line tool to creating and verifying signatures of artifacts stored in an OCI registry. It is most commonly used for container images. See the [section on signing container images with notation](/documentation/signing-containers/notation) for more details.

### Installation

The notation plugin can be installed using the following command:

~~~bash
notation plugin install --url https://download.signpath.io/cryptoproviders/notation/1.0/linux/x64/notation-signpath.tar.gz --sha256sum <TODO>
#or
notation plugin install --file /path/to/downloaded/notation-signpath
~~~

### Usage

* The available [configuration values](/documentation/crypto-providers#crypto-provider-configuration) can also be passed in via the command line arguments `--plugin-config "Key=<Value>"`. 
* The notation _key id_ is comprised of the _project slug_ and _signing policy slug_, separated by a forward slash, e.g. `"MyProject/release-signing"`

~~~bash
export SIGNPATH_API_KEY=...your-api-key...
notation sign \
  --signature-format cose \
  --id "MyProject/release-signing" \
  --plugin signpath \
  --plugin-config "OrganizationId=0241f767-69c8-448d-ad5e-8bd453916068" \
  $IMAGE_IDENTIFIER
~~~

[notation]: https://github.com/notaryproject/notation