---
header: GitLab
layout: resources
toc: true
show_toc: 3
description: GitLab
---

## Prerequisites

* Use the predefined Trusted Build System _GitLab.com_ (see [configuration](/documentation/trusted-build-systems#configuration))
  *  add it to the Organization
  *  link it to each SignPath Project for GitLab

{:.panel.info}
> **Self-managed installations**
>
> SignPath hosts an instance of the GitLab connector which is linked to GitLab.com For integrating self-managed GitLab instances, contact our [support](https://signpath.io/support) team.

## Checks performed by SignPath

The GitLab connector performs the following checks:

* The artifact was built by a GitLab Pipeline, not by some other entity in possession of the API token
* [Origin metadata](/documentation/origin-verification) is provided by GitLab, not the build script, and can therefore not be forged
* The artifact is stored as a GitLab pipeline artifact before it is submitted for signing

## Usage

We provide a `submit-signing-request` component that can be integrated into a GitLab Pipeline:

{% raw %}
```yaml
include:
  - component: gitlab.com/signpath-test-root/components/submit-signing-request@0.1
    inputs:
      stage: sign
      job_name: sign_my_component_a
      api_token_var_name: SIGNPATH_MY_COMPONENT_A_API_TOKEN
      gitlab_access_token_var_name: SIGNPATH_GITLAB_ACCESS_TOKEN
      organization_id: f437cdbb-2ec0-4958-9a85-c2c0cd5dfa1a
      project_slug: MyComponentA
      signing_policy_slug: release-signing
      gitlab_artifact_job_name: build_job
      gitlab_artifact_path: output/my-executable
      wait_for_completion: true
      output_artifact_path: 
      parameters:
        - myparam: myvalue

build_job:
  stage: build
  script:
    - echo "Building some software..."
  artifacts:
    - output/my-executable
```
{% endraw %}

All values can also be provided via environment variables. See the [parameter list](#supported-parameters) for a complete list of all supported inputs. 

### Setups that don't include _Docker Executors_

For all organizations that don't support _Docker Executors_, we provide a CLI tool can be directly invoked. Please contact our [support team](https://signpath.io/support) for details.

## Supported parameters

{%- include render-table.html table=site.data.tables.trusted-build-systems.gitlab-parameters -%}

[user-defined parameters]: /documentation/artifact-configuration/syntax#parameters

## Environment variables for subsequent jobs

The component invocation will publish a dotenv report and make the following environment variables available in subsequent jobs:

* `${PREFIX}_SIGNPATH_SIGNING_REQUEST_ID`: The id of the newly created signing request.
* `${PREFIX}_SIGNPATH_SIGNING_REQUEST_WEB_URL`: The url of the signing request in SignPath.
* `${PREFIX}_SIGNPATH_SIGNED_ARTIFACT_DOWNLOAD_URL`: The url where the signed artifact can be downloaded.

`${PREFIX}` defaults to the capitalized name of the signing job in GitLab (`SIGN` by default).

TODO: the term _Pipeline Integrity_ is already used by GitLab: https://docs.gitlab.com/ci/pipeline_security/#pipeline-integrity
TODO: Update SLSA page also
