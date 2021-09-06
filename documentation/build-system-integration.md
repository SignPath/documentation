---
main_header: Documentation
sub_header: Build System Integration
layout: resources
toc: true
show_toc: 3
redirect_from: /documentation/integrations
description: Documentation for integrating SignPath with Continuous Integration (CI) platforms and pipelines
---

## Abstract

This section describes how SignPath can be integrated into automated builds using continuous integration software. You can use the PowerShell module provided by SignPath, directly call the Web API to submit signing requests, or integrate SignPath as part of your AppVeyor build step.

<div class='panel info' markdown='1' >
<div class='panel-header'>Locating ID values</div>
All necessary IDs can be found on the signing policy details page, including a code snippet that calls the PowerShell module.
</div>

## Authorization

Before accessing the API, you have to create a CI User in the User section of the SignPath application.

The API token is displayed when a new CI user is created. (If you lose the API key, you will need to generate a new one.)

Make sure to keep the access token in a secure location. Most Continuous Integration (CI) systems provide a mechanism to store secrets, which is usually the best place to keep API tokens. If you use several distinct systems for API access, we recommend that you create individual CI User accounts in SignPath.

## PowerShell 

[![PowerShell Gallery](https://img.shields.io/powershellgallery/v/SignPath.svg?style=flat-square&label=PowerShell%20Gallery)](https://www.powershellgallery.com/packages/SignPath/)

To integrate SignPath in your build chain, you can use the [official SignPath module from PowerShell Gallery](https://www.powershellgallery.com/packages/SignPath).

Create signing requests by calling the following commands via PowerShell:

* Install the SignPath module
  ~~~ powershell
  Install-Module -Name SignPath
  ~~~
  <div class="panel tipp" markdown="1">
  <div class="panel-header">Specify an an acceptable version range</div>
  The releases of the SignPath module follow [semantic versioning](https://semver.org/) principles. In automated scenarios, we recommend to fix the major version by specifying a lower and upper bound, e.g. `-MinimumVersion 2.1.0 -MaximumVersion 2.999.999` to ensure that a compatible version is installed.
  </div>
* Submit a signing request and get a signing request ID without waiting for completion ...
  ~~~ powershell
  $signingRequestID = Submit-SigningRequest `
      -CIUserToken $CI_USER_TOKEN `
      -OrganizationId $ORGANIZATION_ID `
      -ProjectSlug $PROJECT `
      -SigningPolicySlug $SIGNING_POLICY `
      -ArtifactConfigurationSlug $ARTIFACT_CONFIGURATION `
      -InputArtifactPath $PATH_TO_INPUT_ARTIFACT
  ~~~ 
* ... and download the signed artifact later
  ~~~ powershell
  Get-SignedArtifact `
      -CIUserToken $CI_USER_TOKEN `
      -OrganizationId $ORGANIZATION_ID `
      -SigningRequestId $signingRequestID `
      -OutputArtifactPath $PATH_TO_OUTPUT_ARTIFACT
  ~~~ 
* Or submit a signing request and wait for completion
  ~~~ powershell
  Submit-SigningRequest `
      -CIUserToken $CI_USER_TOKEN `
      -OrganizationId $ORGANIZATION_ID `
      -ProjectSlug $PROJECT `
      -SigningPolicySlug $SIGNING_POLICY `
      -ArtifactConfigurationSlug $ARTIFACT_CONFIGURATION `
      -InputArtifactPath $PATH_TO_INPUT_ARTIFACT `
      -OutputArtifactPath $PATH_TO_OUTPUT_ARTIFACT `
      -WaitForCompletion
  ~~~

#### User-defined parameters
<span class='badge'><i class='icon-signpath'></i>Available for Enterprise subscriptions</span>

Values for [user-defined parameters](/documentation/artifact-configuration#user-defined-parameters) in the artifact configuration can be provided adding a `Parameters` argument.

Example: `-Parameters @{ productVersion="1.2.0" }`

## HTTP REST API

In case the PowerShell module is not sufficient, you can communicate directly with our API by calling our public HTTP REST endpoints.

### Base URL and authentication

SignPath.io uses bearer authentication.

| Common API arguments      |     |
| ------------------------- | --- |
| Base URL                  | `https://app.signpath.io/API/v1/$(OrganizationId)`
| Authorization HTTP header | `Authorization: Bearer $(token)`


You need to provide these values for every single API request.

### Submit a signing request

| Synopsis                    |      |
| --------------------------- | ---- |
| URL                         | `/SigningRequests`
| Method                      | `POST`
| Encoding                    | `multipart/form-data`
| `ProjectSlug`               | The project for which you want to create the signing request
| `SigningPolicySlug`         | Signing policy for which you want to create the signing request
| `ArtifactConfigurationSlug` | Optional: artifact configuration to use for the signing request (default if not specified)
| `Artifact`                  | Artifact file
| `Description`               | Optional: description for your signing request (e.g. version number)

**Example:**

~~~ bash
curl -H "Authorization: Bearer $CI_USER_TOKEN" \
     -F "ProjectSlug=$PROJECT" \
     -F "SigningPolicySlug=test-signing" \
     -F "ArtifactConfigurationSlug=v2.4" \
     -F "Artifact=@$PATH_TO_ARTIFACT" \
     -F "Description=$DESCRIPTION" \
     https://app.signpath.io/API/v1/$ORGANIZATION_ID/SigningRequests
~~~

**Success result:** HTTP status code `201`. A HTTP `Location` response-header field is returned with the URL of the created entity.

#### User-defined parameters
<span class='badge'><i class='icon-signpath'></i>Available for Enterprise subscriptions</span>

Values for [user-defined parameters](/documentation/artifact-configuration#user-defined-parameters) in the artifact configuration can be provided by adding another multipart/form-data field prefixed with `Parameters.`

Example: `-F "Parameters[productVersion]=1.2.0"`

### Get signing request data

| Synopsis   |      |
| ---------- | ---- |
| URL        | `/SigningRequests/$(SigningRequestId)` <br> (`Location` response-header from the call that submitted the signing request)
| Method     | GET
| Parameters | none

**Example:**

~~~ bash
curl -H "Authorization: Bearer $CI_USER_TOKEN" \
     https://app.signpath.io/API/v1/$ORGANIZATION_ID/SigningRequests/$SIGNING_REQUEST_ID
~~~

**Success result:** HTTP status code `200`. Signing request data in JSON format:

~~~ json
{
  "status":"Completed",
  "isFinalStatus": true,
  "description":"Called by cURL",
  "projectId":"c90eb2c7-d34e-49fc-9e90-c00235ecaf1a",
  "projectSlug":"test-project",
  "projectName":"Test project",
  "artifactConfigurationId":"24b767a6-092f-4104-869d-25f0da159576",
  "artifactConfigurationSlug":"Default",
  "artifactConfigurationName":"Default",
  "signingPolicyId":"137ada35-fc11-4719-a3a4-269983692197",
  "signingPolicySlug":"test-signing",
  "signingPolicyName":"test-signing",
  "unsignedArtifactLink":"https://app.signpath.io/API/v1/c2099ac1-b4b5-4b30-934e-3933c2d9922d/SigningRequests/a4559e13-9e95-480a-9567-5b8a3252bb27/UnsignedArtifact",
  "signedArtifactLink":"https://app.signpath.io1/API/v1/c2099ac1-b4b5-4b30-934e-3933c2d9922d/SigningRequests/a4559e13-9e95-480a-9567-5b8a3252bb27/SignedArtifact",
  "origin": {
    "buildData": {
      "buildSettingsFile": {
        "downloadLink": "https://app.signpath.io/API/v1/c2099ac1-b4b5-4b30-934e-3933c2d9922d/SigningRequests//a4559e13-9e95-480a-9567-5b8a3252bb27/BuildSettingsFile",
        "fileName": "AppVeyorSettings.json"
      },
      "url": "https://ci.appveyor.com/project/TestUser/Test-Project/builds/12345678/job/03rba4p8tlr2t4f7"
    },
    "repositoryData": {
      "url": "https://github.com/name/project",
      "branchName": "master",
      "commitId": "efe8bbc00c5484bfd38ce13a749ea2103a8ea713",
      "sourceControlManagementType": "git"
    }
  },
  "parameters": {
    "param1": "value1",
    "param2": "value2"
  }
}
~~~

* **Possible `status` values:** `WaitingForApproval`, `QueuedForMalwareScanning`, `ScanningForMalware`, `QueuedForProcessing`, `Processing`, `Completed` (final), `Failed` (final), `Denied` (final), `Canceled` (final), `RetrievingArtifact`, `ArtifactRetrievalFailed` (final)
* `origin` is only available for signing requests with origin verification

### Download the signed artifact

Once the signing request is successfully completed, the status response contains a `signedArtifactLink` field with a link to the signed artifact file. It can easily be retrieved by issuing the following command:

| Synopsis   |      |
| ---------- | ---- |
| URL        | `/SigningRequests/$(SigningRequestId)/SignedArtifact` <br> (`signedArtifactLink` field from `GET SigningRequests/`id)
| Method     | GET  |
| Parameters | none

**Example:**

~~~ bash
curl -H "Authorization: Bearer $CI_USER_TOKEN" \
     -o $LOCAL_PATH_TO_DOWNLOADED_ARTIFACT \
     https://app.signpath.io/API/v1/$ORGANIZATION_ID/SigningRequests/$SIGNING_REQUEST_ID/SignedArtifact
~~~

**Success result:** HTTP status code `200`. Returns the binary content of the signed artifact.

## CI integrations with origin verification

Origin verification ensures that a signed artifact is the result of building a specific source code version.

In order to achieve this, you cannot submit a signing request from an unfinished build. Instead, you have to finish the build job without signing, and then trigger a signing request with origin verification. 

When signing is completed, you can use a Webhook handler for further processing, such as uploading the signed artifact to a repository. 

Origin verification results in additional confidence. It provides the foundation for restricted Open Source signing.

Origin verification verifies the following information:

* **Source code repository URL** as specified in the SignPath **project**
* **Branch** as specified in the **signing policy**
* **Commit version**
* **Build job URL** as provided by the CI system
* **Reproducability** checks that the build process is completely determined by the source code repository

Verification of reproducability depends on the CI system used. Typical verifications include:

* Build settings are fully determined by a configuration file under source control
* No manual overrides of critical build settings in CI system's build job
* Prevent caching from previous (unverified) builds

The final goal of origin verification is enable signing policies based on source code reviews. 

<div class="panel warning" markdown="1">
<div class="panel-header">Source code reviews must include build scripts</div>
Note that a build script or makefile can download any software from the Internet and include it as a build artifact. SignPath cannot possibly detect or prevent this, but it can make sure that any such evasion will be visible in the source code repository.

Make sure that your source code review policy includes build scripts and makefiles.
</div>

<div class="panel tipp" markdown="1">
<div class="panel-header">Use origin verification restrictions</div>
Enable additional restrictions for signing policies that use release certificates:

* Select **Verify origin** to make sure that only verified builds can be signed
* Define source code review policies for branches that are supposed to be used for production releases. Use the **Allowed branch names** setting to make sure that a signing policy can only be used for those branches. Typical settings include `master` or `release/*`.
* If you need to be able to sign other builds under special circumstances, consider adding another signing policy with strong approval requirements (e.g. 2 out of *n*).
</div>

### AppVeyor

#### Prerequisites and restrictions

This feature is currently dedicated to Open Source projects. Future versions will allow disabling certain limitations in paid subscriptions.

Current limitations:

* The AppVeyor project and the Git repository must be public 

The following checks are performed:

* No additional scripts may be executed during the build step and no cache entries may be used (so that the build remains fully traceable and is only built from the repository). The scripts must be all set to off at the bottom of the project settings page on AppVeyor.
* The build settings may not be modified between starting the AppVeyor build and calling SignPath.io
* The build configuration must be stored in the root directory under the name `appveyor.yml` or `.appveyor.yml` (no custom name is allowed to be set under *Project settings* and *Custom configuration .yml file name*)

This is to ensure that the binary artifacts result purely from the specified source code.

<div class="panel todo" markdown="1">
<div class="panel-header">TODO</div>

Is the following list complete? see https://www.appveyor.com/docs/build-configuration/#generic-git-repositories-and-yaml
> At the moment those supported are: GitHub (hosted and on-premises), Bitbucket (hosted and on-premises), GitLab (hosted and on-premises), Azure DevOps, Kiln and Gitea. 
</div>


#### Setup
This figure shows the secrets that must be shared between AppVeyor.com and SignPath.io:
![AppVeyor Setup flow](/assets/img/resources/documentation_build-integration_appveyor.png)

<table style="table-layout: auto;">
<thead>
  <tr>
    <th style="width: 20%;">Action</th>
    <th style="width: 60%;">Steps</th>
    <th style="width: 20%;">Remarks</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Add an AppVeyor integration to a SignPath project</td>
    <td markdown="1">

1. On [ci.appveyor.com](https://ci.appveyor.com)
   * Select *Accuont* and *Security*
   * Make sure the checkboxes for both *API v1* and *API v2* are checked
2. On [ci.appveyor.com](https://ci.appveyor.com)
   * Select *My Profile* and *API Keys*
   * Remember the **Bearer token** for the next step
3. On [SignPath.io](https://app.signpath.io)
   * Navigate to your *project*, go to *Trusted Build Systems* and add a link to *AppVeyor*
   * In the dialog, enter the **API key** you just acquired

</td>
    <td>SignPath.io must authenticate against Appveyor to retrieve the build artifacts</td>
  </tr> <tr>
    <td>Encrypt the SignPath API token in AppVeyor</td>
    <td markdown="1">

1. On [SignPath.io](https://app.signpath.io)
   * Choose the Users menu and create a new *CI User* or open an existing one
   * Remember the **SignPath API token** for the next step
2. On [ci.appveyor.com](https://ci.appveyor.com)
   * Open *Account Settings* and choose *[Encrypt YAML](https://ci.appveyor.com/tools/encrypt)*
   * Enter **``Bearer <SIGNPATH_API_TOKEN>``** (without &lt;brackets&gt;)
   * Remember the **encrypted SignPath API token** for the next step

</td>
    <td>AppVeyor lets you encrypt secret values. You can then safely use the encrypted string in your appveyor.yaml file</td>
  </tr> <tr>
    <td>Add a deploy Webhook</td>
    <td colspan="2" markdown="1">

Append this to your appveyor.yaml file:

~~~ yaml
deploy:
- provider: Webhook
  url: https://app.signpath.io/API/v1/<ORGANIZATION_ID>/Integrations/AppVeyor?ProjectSlug=<PROJECT>&SigningPolicySlug=<SIGNING_POLICY>
  authorization:
     secure: <ENCRYPTED_SIGNPATH_API_TOKEN>
~~~

Replace the parameters:
* `<ORGANIZATION_ID>`, `<PROJECT>` and `<SIGNING_POLICY>` values can be retrieved from the signing policy page
* `<ENCRYPTED_SIGNPATH_API_TOKEN>` is the value from the previous step

</td> </tr> </tbody> </table>

#### Attached build documentation

SignPath adds the following information to packages:

* For NuGet packages:
  1. The build settings are stored in an AppVeyorSettings.json file in the root of the NuGet package
  2. The commit hash and repository URL are written to the metadata of the NuGet package

These steps allow consumers of the signed artifact to verify source code version and build settings.

## Other CI integrations

### Azure DevOps

[![Azure DevOps installations](https://img.shields.io/visual-studio-marketplace/azure-devops/installs/total/SignPath.signpath-tasks?color=blue&label=Visual+Studio+Marketplace+installs)](https://marketplace.visualstudio.com/items?itemName=SignPath.signpath-tasks)

For Azure DevOps, you can use build pipeline tasks from the [official SignPath extension on Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SignPath.signpath-tasks).

## Webhook notifications

If you want to react on completed signing requests, you can add a **Webhook** **notification** in your project's **Integration** section.

For each completed signing request, SignPath will post the following JSON information to the specified URL:

~~~ json
{
  "OrganizationId": "094f5736-6b8c-4ca7-9514-0933c8b928e2",
  "SigningRequestId": "b4596ce6-1b8d-4527-9cce-16b3e174fb3d",
  "Status": "Completed"
}
~~~

A handler for this Webhook can use the Web API for further activities, such as pushing the signed artifact to a repository. Use the Web API to get signing request data including build information.

<div class="panel info" markdown="1">
<div class="panel-header">Webhook authentication</div>

Webhooks will send the `Authentication` header exactly as specified. Don't forget to add the method, for example:

`Bearer JEAG1OrTXZ/t4URp5URt40DLBlA3WtcJmbwfeosyBkTABr6r`
</div>
