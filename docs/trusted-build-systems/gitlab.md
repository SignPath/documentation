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
* Specify `<zip-file>` as root element of your [Artifact Configurations](/documentation/artifact-configuration) (GitLab packages all artifacts as ZIP archives)

{:.panel.info}
> **Self-managed installations**
>
> SignPath hosts an instance of the GitLab connector which is linked to GitLab.com For integrating self-managed GitLab instances, contact our [support](/support) team.

## Checks performed by SignPath

The GitLab connector performs the following checks:

* The artifact was built by a GitLab Pipeline, not by some other entity in possession of the API token
* [Origin metadata](/documentation/origin-verification) is provided by GitLab, not the build script, and can therefore not be forged
* The artifact is stored as a GitLab pipeline artifact before it is submitted for signing

## Usage

There are two ways to integrate SignPath into your GitLab Pipeline:

* If your GitLab instance has at least one runner with a _Docker Executor_ (like GitLab.com), we recommend using the _SignPath component_
* If your self-managed GitLab instance does not support runners with _Docker Executors_, you can still download and call our command-line tool manually

### SignPath component

We provide a `submit-signing-request` component that can be integrated into a GitLab Pipeline:

{% raw %}
```yaml
include:
  - component: gitlab.com/signpath/components/submit-signing-request@0.1
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

### `signpath-gitlab` CLI tool

For all organizations that don't support _Docker Executors_, the `signpath-gitlab` CLI tool can be directly invoked:

{% raw %}
```yaml
stages:
  - build
  - sign

build_job:
  stage: build
  script:
    - echo "Building some software..."
  artifacts:
    - output/my-executable

sign_job:
  stage: sign
  script:
    - curl -o signpath-gitlab -L https://download.signpath.io/ci-integrations/gitlab/0.1/linux/x64/signpath-gitlab
    - |
      ./signpath-gitlab submit-signing-request \
        --api-token $SIGNPATH_API_TOKEN \
        --gitlab-access-token $SIGNPATH_GITLAB_ACCESS_TOKEN \
        --organization-id $SIGNPATH_ORGANIZATION_ID \
        --project-slug MyProject \
        --signing-policy-slug release-signing \
        --gitlab-artifact-job-name build_job \
        --gitlab_artifact_path: output/app.exe \
        --wait-for-completion true \
        --output-artifact-path signed-output \
        --parameters '{ "myparam": "myvalue" }'
  artifacts:
    - signed-output
```
{% endraw %}

All values can also be provided via environment variables.  See the [parameter list](#supported-parameters) for a complete list of all supported parameters.

### Supported parameters

The parameters can be passed to the

* component via _inputs_ or _environment variables_
* CLI tool via _arguments_ or _environment variables_

{%- include render-table.html table=site.data.tables.trusted-build-systems.gitlab-parameters -%}

[user-defined parameters]: /documentation/artifact-configuration/syntax#parameters

### Environment variables for subsequent jobs

The component invocation will publish a dotenv report and make the following environment variables available in subsequent jobs:

* `${PREFIX}_SIGNPATH_SIGNING_REQUEST_ID`: The id of the newly created signing request.
* `${PREFIX}_SIGNPATH_SIGNING_REQUEST_WEB_URL`: The url of the signing request in SignPath.
* `${PREFIX}_SIGNPATH_SIGNED_ARTIFACT_DOWNLOAD_URL`: The url where the signed artifact can be downloaded.

`${PREFIX}` defaults to the capitalized name of the signing job in GitLab (`SIGN` by default).

TODO: the term _Pipeline Integrity_ is already used by GitLab: https://docs.gitlab.com/ci/pipeline_security/#pipeline-integrity
TODO: Update SLSA page also