# Authenticating with OpenID Connect
Follow the steps below to authenticate with [Open ID Connect](https://www.microsoft.com/security/business/security-101/what-is-openid-connect-oidc):

1. [Create a Microsoft Entra application and service principal](https://learn.microsoft.com/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#create-a-microsoft-entra-application-and-service-principal)
1. [Add federated credentials](https://learn.microsoft.com/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#add-federated-credentials)
1. [Create GitHub secrets](https://learn.microsoft.com/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#create-github-secrets)

1. Assign the [Trusted Signing Certificate Profile Signer](https://learn.microsoft.com/azure/trusted-signing/concept-trusted-signing-resources-roles#supported-roles) role to your service principal.
   1. Open your Trusted Signing Account in the Azure portal.
      1. **Note:** You can assign the role from your Resource Group or Subscription if you have multiple Trusted Signing accounts.
   1. Navigate to the Access Control (IAM) tab.
   1. Click 'Add role assignment'.
   1. Select 'Trusted Signing Certificate Profile Signer'.
   1. Next.
   1. Assign access to your 'User, group, or service principal' or 'Managed identity'.
   1. Review + assign.

1. Adapt the following yaml to your GitHub pipeline:
    ```yaml
    permissions:
      id-token: write
      contents: read

    jobs:
      sign:
        runs-on: windows-latest

        steps:
          - name: Azure login
            uses: azure/login@v1
            with:
              client-id: ${{ secrets.AZURE_CLIENT_ID }}
              tenant-id: ${{ secrets.AZURE_TENANT_ID }}
              subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

          - name: Trusted Signing
            uses: azure/trusted-signing-action@vx.y.z
            with:
              ...
              exclude-environment-credential: true
              exclude-workload-identity-credential: true
              exclude-managed-identity-credential: true
              exclude-shared-token-cache-credential: true
              exclude-visual-studio-credential: true
              exclude-visual-studio-code-credential: true
              exclude-azure-cli-credential: false
              exclude-azure-powershell-credential: true
              exclude-azure-developer-cli-credential: true
              exclude-interactive-browser-credential: true
    ```
