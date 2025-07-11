<p align="center">
  <img src="icon.png" alt="Trusted Signing" />
</p>

# Trusted Signing
The Trusted Signing Action allows you to digitally sign your files using a Trusted Signing certificate during a GitHub Actions run.

## Runner Requirements
This Action can only be executed on Windows runners. It is supported by the following GitHub hosted runners:
- [windows-2022](https://github.com/actions/runner-images/blob/main/images/windows/Windows2022-Readme.md)
- [windows-2019](https://github.com/actions/runner-images/blob/main/images/windows/Windows2019-Readme.md)

It is also possible to use self-hosted runners with the following requirements:
- Windows 7+
- PowerShell 5.1+
- .NET runtime 8.0+

## Example
The example below shows how to sign the build output of a simple Wpf application.

```yaml
on:
  push:
    branches: [main]

jobs:
  build-and-sign:
    runs-on: windows-latest
    name: Build app and sign files with Trusted Signing
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: dotnet restore App

      - name: Build
        run: dotnet build --configuration Release --no-restore WpfApp

      - name: Sign files with Trusted Signing
        uses: azure/trusted-signing-action@v0
        with:
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
          endpoint: https://eus.codesigning.azure.net/
          trusted-signing-account-name: vscx-codesigning
          certificate-profile-name: vscx-certificate-profile
          files-folder: ${{ github.workspace }}\App\App\bin\Release\net8.0-windows
          files-folder-filter: exe,dll
          file-digest: SHA256
          timestamp-rfc3161: http://timestamp.acs.microsoft.com
          timestamp-digest: SHA256
```

## Usage
Please review [Common error codes and mitigations](https://learn.microsoft.com/azure/trusted-signing/faq#common-error-codes-and-mitigations) before filing an issue.

### Authentication
Behind the scenes, the Action uses [DefaultAzureCredential](https://learn.microsoft.com/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) as the primary method of authentication to Azure. The [EnvironmentCredential](https://learn.microsoft.com/dotnet/api/azure.identity.environmentcredential?view=azure-dotnet) variables are exposed as inputs and then set to Action-scoped environment variables. Each credential type supported by `DefaultAzureCredential` can be disabled using the Action inputs.

> [!NOTE]
> [Trusted Signing Certificate Profile Signer](https://learn.microsoft.com/azure/trusted-signing/concept-trusted-signing-resources-roles#supported-roles) role is required to successfully sign with Trusted Signing

It is reccomended to use OpenID Connect for authentication with the Trusted Signing service. See [Authenticating with OpenID Connect](docs/OIDC.md) for details on how to configure your GitHub pipeline with OIDC and Federated Credentials.

#### App Registration
```yaml
# The Azure Active Directory tenant (directory) ID.
azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}

# The client (application) ID of an App Registration in the tenant.
azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}

# A client secret that was generated for the App Registration.
azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
```

#### Azure Active Directory
```yaml
# The username, also known as upn, of an Azure Active Directory user account.
azure-username: ${{ secrets.AZURE_USERNAME }}

# The password of the Azure Active Directory user account. Note this does not support accounts with MFA enabled.
azure-password: ${{ secrets.AZURE_PASSWORD }}
```

#### Exclude Credentials
```yaml
# Exclude the "EnvironmentCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-environment-credential: false

# Exclude the "WorkloadIdentity" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-workload-identity-credential: false

# Exclude the "ManagedIdentity" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-managed-identity-credential: false

# Exclude the "SharedTokenCacheCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-shared-token-cache-credential: false

# Exclude the "VisualStudioCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-visual-studio-credential: false

# Exclude the "VisualStudioCodeCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-visual-studio-code-credential: false

# Exclude the "AzureCliCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-azure-cli-credential: false

# Exclude the "AzurePowerShellCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-azure-powershell-credential: false

# Exclude the "AzureDeveloperCliCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is false.
exclude-azure-developer-cli-credential: false

# Exclude the "InteractiveBrowserCredential" type from being considered when authenticating with "DefaultAzureCredential". The default value is true.
exclude-interactive-browser-credential: true
```

### Account Details
```yaml
# The Trusted Signing Account endpoint. The URI value must have a URI that aligns to the region your Trusted Signing Account and Certificate Profile you are specifying were created in during the setup of these resources.
endpoint: https://eus.codesigning.azure.net/

# The Trusted Signing Account name.
trusted-signing-account-name: my-account-name

# The Certificate Profile name.
certificate-profile-name: my-profile-name
```

### File Specification
#### Files List
This strategy allows you to specify a comma separated list of files to be signed.

```yaml
# A comma or newline separated list of absolute paths to the files being signed. Can be combined with the files-folder and file-catalog inputs.
files: ${{ github.workspace }}\files\app.dll,${{ github.workspace }}\files\app.exe

files: |
  ${{ github.workspace }}\files\app.dll
  ${{ github.workspace }}\files\app.exe
```

#### Files Folder
This strategy allows you to specify a folder that contains all the files you want signed. There are options available for narrowing the focus as well. For example, you can use the `files-folder-filter` input to specify that you only want `exe` files to be signed. See [about_Wildcards](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_wildcards) for more information on using wildcards with the `files-folder-filter` input.

```yaml
# The folder containing files to be signed. Can be combined with the file-catalog input.
files-folder: ${{ github.workspace }}\App\App\bin\Release\net6.0-windows

# A comma separated list of file extensions that determines which types of files will be signed in the folder specified by the files-folder input. Any file type not included in this list will not be signed. If this input is not used, all files in the folder will be signed. Supports wildcards for matching multiple file names with a pattern.
files-folder-filter: dll,exe,msix

# A boolean value (true/false) that indicates if the folder specified by the files-folder input should be searched recursively. The default value is false.
files-folder-recurse: true

# An integer value that indicates the depth of the recursive search toggled by the files-folder-recurse input. By default there is no limit to the depth of the search.
files-folder-depth: 2
```

Given the following directory structure:
```txt
C:.
└───files
        System.dll
        Foo.Bar.Core.dll
        Foo.Bar.Utilities.dll
        Foo.Bar.exe
        LICENSE.md
```

Here is an example of inputs that can be used to specify that only the `Foo.Bar.*` files are signed:
```yaml
files-folder: ${{ github.workspace }}\files
files-folder-filter: Foo.Bar.*.dll,*.exe
files-folder-recurse: false
files-folder-depth: 1
```

#### Files Catalog
This strategy allows you to specify a precise list of files to be signed.

```yaml
# A file containing a list of relative paths to the files being signed. The paths should be relative to the location of the catalog file. Each file path should be on a separate line. Can be combined with the files-folder input.
files-catalog: ${{ github.workspace }}\catalog.txt
```

Given the following directory structure:
```txt
C:.
│   catalog.txt
│
└───files
        System.dll
        Foo.Bar.Core.dll
        Foo.Bar.Utilities.dll
        Foo.Bar.exe
        LICENSE.md
```

Here is an example of a `catalog.txt` file that can be used to specify that only the `Foo.Bar.*` files are signed:
```txt
./files/Foo.Bar.Core.dll
./files/Foo.Bar.Utilities.dll
./files/Foo.Bar.exe
```

### Digest Algorithm
```yaml
# The name of the digest algorithm used for hashing the file being signed. The supported values are SHA256, SHA384, and SHA512. The default value is SHA256.
file-digest: SHA256
```

### Timestamping
```yaml
# A URL to an RFC3161 compliant timestamping service.
timestamp-rfc3161: http://timestamp.acs.microsoft.com

# The name of the digest algorithm used for timestamping. The supported values are SHA256, SHA384, and SHA512.
timestamp-digest: SHA256
```

### Append Signature
```yaml
# A boolean value (true/false) that indicates if the signature should be appended. If no primary signature is present, this signature is made the primary signature instead. The default value is false.
append-signature: true
```

### Description
```yaml
# A description of the signed content.
description: My signed content.

# A Uniform Resource Locator (URL) for the expanded description of the signed content.
description-url: https://github.com/azure/trusted-signing-action
```

### Digest
```yaml
# Generates the digest to be signed and the unsigned PKCS7 files at this path. The output digest and PKCS7 files will be path\FileName.dig and path\FileName.p7u. To output an additional XML file, see `generate-digest-xml`.
generate-digest-path: ${{ github.workspace }}\digest

# A boolean value (true/false) that indicates if an XML file is produced when the `generate-digest-path` input is used. The output file will be Path\FileName.dig.xml. The default value is false.
generate-digest-xml: true

# Creates the signature by ingesting the signed digest to the unsigned PKCS7 file at this path. The input signed digest and unsigned PKCS7 files should be path\FileName.dig.signed and path\FileName.p7u.
ingest-digest-path: ${{ github.workspace }}\digest

# A boolean value (true/false) that indicates if only the digest should be signed. The input file should be the digest generated by the `generate-digest-path` input. The output file will be File.signed. The default value is false.
sign-digest: true
```

### Pages Hashes
```yaml
# A boolean value (true/false) that indicates if page hashes should be generated for executable files. The default value is false.
generate-page-hashes: true

# A boolean value (true/false) that indicates if page hashes should be suppressed for executable files. The default is determined by the SIGNTOOL_PAGE_HASHES environment variable and by the wintrust.dll version. This input is ignored for non-PE files.
suppress-page-hashes: true
```

### PKCS7
```yaml
# A boolean value (true/false) that indicates if a Public Key Cryptography Standards (PKCS) #7 file is produced for each specified content file. PKCS #7 files are named path\filename.p7. The default value is false.
generate-pkcs7: true

# Options for the signed PKCS #7 content. Set value to "Embedded" to embed the signed content in the PKCS #7 file, or to "DetachedSignedData" to produce the signed data portion of a detached PKCS #7 file. If this input is not used, the signed content is embedded by default.
pkcs7-options: Embedded

# The object identifier (OID) that identifies the signed PKCS #7 content.
pkcs7-oid: 1.3.6.1.5.5.7.3.3
```

### Enhanced Key Usage
```yaml
# The enhanced key usage (EKU) that must be present in the signing certificate. The usage value can be specified by OID or string. The default usage is "Code Signing" (1.3.6.1.5.5.7.3.3).
enhanced-key-usage: 1.3.6.1.5.5.7.3.3
```

### ClickOnce
```yaml
# The application name for any ClickOnce files being signed.
clickonce-application-name: My ClickOnce application name.

# The publisher name for any ClickOnce files being signed.
clickonce-publisher-name: My ClickOnce publisher name.
```

### Miscellaneous
```yaml
# The number of seconds that the Trusted Signing service will wait for all files to be signed before it exits. The default value is 300 seconds.
timeout: 600

# The summed length of file paths that can be signed with each signtool call. This parameter should only be relevant if you are signing a large number of files. Increasing the value may result in performance gains at the risk of potentially hitting your system's maximum command length limit. The minimum value is 0 and the maximum value is 30000. A value of 0 means that every file will be signed with an individual call to signtool.
batch-size: 10000

# A boolean value (true/false) that indicates if the dependencies for this action should be cached by GitHub or not. The default value is true. When using self-hosted runners, caches from workflow runs are stored on GitHub-owned cloud storage. A customer-owned storage solution is only available with GitHub Enterprise Server. When enabled, this option can reduce the duration of the action by at least 1 minute. More info: https://docs.github.com/actions/using-workflows/caching-dependencies-to-speed-up-workflows
cache-dependencies: true

# A boolean value (true/false) that controls trace logging. The default value is false.
trace: false

# A unique identifier for the signing request. This value is used to track the signing request in the Trusted Signing service.
correlation-id: aaaa0000-bb11-2222-33cc-444444dddddd
```

## Best Practices
### ClickOnce
Generally you will want to sign an entire package and all its contents i.e. the deployment manifest (`.application` or `.vsto`), application manifest (`.exe.manifest` or `.dll.manifest`) and the underlying `.exe` and `.dll` files themselves. To do this, ensure that the entire contents of the package are available (i.e. the whole `publish` folder from your build) and pass the deployment manifest (`.application` or `.vsto`) as the file to sign - the rest of the files will be detected and signed in the proper order automatically.

In the example below, it is only necessary to pass `ClickOnceApp.application` and `setup.exe` to the Trusted Signing Action. The remaining "Application Files" will be signed automatically.

```txt
C:\TEST\ASSETS\SAMPLE-FILES\CLICKONCE
│   ClickOnceApp.application
│   setup.exe
└───Application Files
    └───ClickOnceApp_1_0_0_0
            ClickOnceApp.deps.json.deploy
            ClickOnceApp.dll.deploy
            ClickOnceApp.dll.manifest
            ClickOnceApp.exe.deploy
            ClickOnceApp.runtimeconfig.json.deploy
            Launcher.exe.deploy
```

The following inputs are ignored when signing ClickOnce files:
- `append-signature`
- `generate-digest-path`
- `generate-digest-xml`
- `ingest-digest-path`
- `sign-digest`
- `generate-page-hashes`
- `suppress-page-hashes`
- `generate-pkcs7`
- `pkcs7-options`
- `pkcs7-oid`
- `enhanced-key-usage`

### Timestamping
The files must be signed with timestamping enabled in order for the signatures to be valid for longer than 3 days. It is recommended to use the Trusted Signing timestamp server:
```yaml
timestamp-rfc3161: 'http://timestamp.acs.microsoft.com'
timestamp-digest: 'SHA256'
```

### Authentication
This Action performs authentication using [DefaultAzureCredential](https://learn.microsoft.com/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) which attempts a series of authentication methods in order. If one method fails, it will attempt the next one until authentication is successful.

Each authentication method can be [disabled individually](https://github.com/Azure/trusted-signing-action#exclude-credentials) so that no time is wasted attempting to authenticate with methods that will never pass.

For example, when authenticating with [EnvironmentCredential](https://learn.microsoft.com/dotnet/api/azure.identity.environmentcredential?view=azure-dotnet) specifically, disable the other credentials with the following inputs:
```yaml
exclude-environment-credential: false
exclude-workload-identity-credential: true
exclude-managed-identity-credential: true
exclude-shared-token-cache-credential: true
exclude-visual-studio-credential: true
exclude-visual-studio-code-credential: true
exclude-azure-cli-credential: true
exclude-azure-powershell-credential: true
exclude-azure-developer-cli-credential: true
exclude-interactive-browser-credential: true
```

This can make the Action fail faster if for some reason the [EnvironmentCredential](https://learn.microsoft.com/dotnet/api/azure.identity.environmentcredential?view=azure-dotnet) fails. Similarly, if using for example an [AzureCliCredential ](https://learn.microsoft.com/dotnet/api/azure.identity.azureclicredential?view=azure-dotnet), then we want to skip over attempting to authenticate with the several methods that come before it in order.

### Self-Hosted Agents
When running the Trusted Signing Action on a self-hosted agent/private runner you may encounter the following error:

`Exception calling "ShouldContinue" with "2" argument(s): "Windows PowerShell is in NonInteractive mode. Read and Prompt functionality is not available."`

This error is caused by the PowerShell NuGet package provider not being installed. By default, Windows PowerShell does not have these components installed.

To resolve this issue, install the NuGet package provider on the agent/runner with the following script:

```yaml
if ((Get-PackageProvider -Name NuGet -ErrorAction Ignore) -eq $Null) {
    Install-PackageProvider -Name NuGet -Force -Scope CurrentUser;
}
```

Additionally, consider installing the latest version of PowerShellGet with the following script:

```yaml
if ((Get-InstalledModule -Name PowerShellGet -MinimumVersion 2.2.1 -ErrorAction Ignore) -eq $Null) {
    Install-Module -Name PowerShellGet -MinimumVersion 2.2.1 -Scope CurrentUser -Force -AllowClobber;
}
```

## Contributing
This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks
This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
