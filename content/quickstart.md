---
title: "Quickstart"
weight: -1
icon: "person-running-regular"
description: "Create your first Upbound Managed Control Plane and connect it to your cloud provider."
---

Upbound is a global platform for building, deploying, and operating cloud
platforms using managed control planes.

## The value of Upbound

As your infrastructure scales, it can be difficult to deal with tech sprawl in a
cloud native world. By offering abstract APIs, Upbound simplifies complex
infrastructure management, making it more accessible for development teams.

Upbound is useful for companies that need a unified control and management
system for their diverse cloud infrastructures. Upbound stands out by providing
fully managed control planes, a unified operating model for extensive management
across various cloud services, and GitOps-driven workflows. This combination of
features allows organizations to efficiently manage their cloud footprint,
streamline developer operations, and maintain consistency across different cloud
environments.

## Prerequisites

You need the following:

- An Upbound account.
- An AWS, Azure, or GCP account with permissions to manage IAM policies.
- A GitHub account with permission to install GitHub Apps.

{{< hint "tip" >}}
If you don't have an Upbound account, [sign up for a free trial](https://accounts.upbound.io/register).
{{< /hint >}}

## Get started

This quickstart guides you through how to create your first managed control
plane in Upbound. Connect Upbound to your cloud provider, and use your control plane to create
and manage infrastructure.

After you register your Upbound account, walk through the interactive "Get
Started" demo below.

<div style="position: relative; padding-bottom: calc(75.92682926829268% + 42px); height: 0;"><iframe src="https://app.supademo.com/embed/clt694nsi0bz4pez12bste3s2" allow="clipboard-write" frameborder="0" webkitallowfullscreen="true" mozallowfullscreen="true" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

## Connect to your cloud provider with OpenID Connect

{{< tabs >}}

{{< tab "AWS" >}}

While Upbound creates your control plane, connect Upbound to AWS.

Upbound recommends using OpenID Connect (OIDC) to authenticate to AWS without exchanging any private information.

Create an [AWS IAM Role](https://console.aws.amazon.com/iamv2/home#/roles) with a **Custom trust policy** for the OIDC connector.

{{<hint "important" >}}
You can find your AWS account ID by selecting the account dropdown in the upper right corner of the AWS console.
{{< /hint >}}

Provide your [AWS account ID](https://docs.aws.amazon.com/signin/latest/userguide/console_account-alias.html), Upbound organization and control plane names in the JSON Policy below.

{{< editCode >}}

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$@YOUR_AWS_ACCOUNT_ID$@:oidc-provider/proidc.upbound.io"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "proidc.upbound.io:sub": "mcp:$@ORG_NAME/CONTROL_PLANE_NAME$@:provider:provider-aws",
          "proidc.upbound.io:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```

{{< /editCode >}}

Follow along with the demo below:

<div style="position: relative; padding-bottom: calc(102.09375% + 42px); height: 0;"><iframe src="https://app.supademo.com/embed/jh_ovMi75LKj8MHOKZzVD" allow="clipboard-write" frameborder="0" webkitallowfullscreen="true" mozallowfullscreen="true" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

{{< /tab >}}

{{< tab "Azure" >}}

While Upbound creates your control plane, connect Upbound to Azure.

Upbound recommends using OpenID Connect (OIDC) to authenticate to Azure without exchanging any private information.

#### Create an identity pool
1. Open the **[Azure portal](https://portal.azure.com/)**.
2. Select **[Azure Active Directory](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview)**.
3. If this is your first time registering Upbound as an identity provider in Azure Active Directory, select **App registrations**
4. At the top of the page, select **New registration**.
5. Name the pool, like **upbound-oidc-provider**.
6. In the _Supported account types_ section select **Accounts in this organizational directory only**.
7. In the _Redirect URI_ section select **Web** and leave the URL field blank.
8. Select **Register**.
{{<img src="images/azure-identity-start.png" alt="Upbound Get Started Configure OIDC screen for Azure"  align="center" unBlur="true" size="small" >}}

#### Create a federated credential

To allow the `upbound-oidc-provider` registration created in the previous step to trust the Upbound Control Plane, do the following in the resource view.

1. Select **Certificates and secrets** in the left navigation.
2. Select **Federated credentials** tab.
3. Select **Add credential**.
4. In _Federated credential scenario_ select **Other Issuer**.
5. In _Issuer_ enter **https://proidc.upbound.io**.
6. In _Subject identifier_ enter:

{{< editCode >}}
```yaml
mcp:$@<your-org>/<your-control-plane-name>$@:provider:provider-azure
```
{{< /editCode >}}

7. In _Credential details name_ enter:

{{< editCode >}}
```yaml
upbound-$@<your-org>-<your-control-plane-name>$@-provider-azure
```
{{< /editCode >}}

8. In _Credential details description_ enter:

{{< editCode >}}
```yaml
upbound MCP $@<your-org>/<your-control-plane-name>$@ Provider provider-azure
```
{{< /editCode >}}

9. Leave _Audience_ unmodified with **api://AzureADTokenExchange**.
10. Select **Add**.

{{<img src="images/azure-registration-config.png" alt="Azure configure app registration" align="center" size="small" unBlur="true">}}

#### Grant permissions to the service principal

For your control plane to be able to perform actions required by this configuration, you need to grant permissions to the Application Service Principal. Assign a role to the Application Service Principal by following instructions at Assign a role to the application.

1. Open the **[Azure portal](https://portal.azure.com/)**
2. Select **[Subscriptions](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade)**.
3. Select your subscription.
4. Select **Access control (IAM)** in the left navigation.
5. Select **Add** and select **Add role assignment**.
6. Find and select the **Contributor** role on the **Privileged administrator roles** tab.
7. Select **Next**.
8. In _Assign access to_ select **User, group, or service principal**.
9. Select **Select members**.
10. Find your application by entering **upbound-oidc-provider** in the search field.
11. Select **Select**.
12. Select **Review + assign**.
13. Make sure everything is correct and press **Review + assign** again.

{{<img src="images/azure-identity-setup.png" alt="Azure grant permissions to service principal" align="center" size="small" unBlur="true" >}}

### Finish configuring the Upbound identity provider

Back in Upbound, finish configuring the identity provider.

In the _Application (client) ID_ field enter your **Application (client) ID**.

For the _Directory (tenant) ID_ field, enter your **Directory (tenant) ID**. You can find this by selecting your Application under Azure Active Directory -> Application Registrations.

In the _Azure Subscription ID_ field, enter your **Subscription ID**. You can find this by selecting your Subscription in the Azure portal.

{{<img src="images/azure-oidc-finalize.png" alt="Upbound configuration to connect to Azure with OIDC" size="small" align="center" unBlur="true">}}

Select **Authenticate**.
Select **Launch Control Plane**.
{{< /tab >}}

{{< tab "GCP" >}}

While Upbound creates your control plane, connect Upbound to GCP.

Upbound recommends using OpenID Connect (OIDC) to authenticate to GCP without exchanging any private information.

{{< hint "warning" >}}
GCP doesn't authenticate a second OIDC pool in the same project connecting to Upbound.

Add a new [Service Account]({{<ref "#create-a-gcp-service-account" >}}) to the existing pool instead.
{{< /hint >}}

#### Create an identity pool
1. Open the **[GCP IAM Admin console](https://console.cloud.google.com/iam-admin/iam)**.
2. Select **[Workload Identity Federation](https://console.cloud.google.com/iam-admin/workload-identity-pools)**.
3. If this is your first Workload Identity Federation configuration select **Get Started**
4. At the top of the page, select **Create Pool**.
5. Name the pool, like **upbound-oidc-pool**.
6. Enter a description like **An identity provider for Upbound**.
7. **Enable** the pool.
8. Select **Continue**
{{<img src="images/gcp-identity-start.png" alt="Upbound Get Started Configure OIDC screen for GCP" align="center" unBlur="true" size="small" >}}

#### Add Upbound to the pool

Under the _Add a provider to pool_ configuration under _Select a provider_ use **OpenID Connect (OIDC)**

_Provider Name_: **upbound-oidc-provider**
_Provider ID_: **upbound-oidc-provider-id**
_Issuer (URL)_: **https://proidc.upbound.io**

Select **Allowed audiences**
For _Audience 1_ enter **sts.googleapis.com**

Select **Continue**.

{{<img src="images/gcp-provider-to-pool.png" alt="GCP add a provider to pool configuration" align="center" size="small" unBlur="true" >}}

#### Configure provider attributes
The provider attributes restrict which remote entities you allow access to your resources.
When Upbound authenticates to GCP it provides an OIDC subject (`sub`) in the form:

`mcp:<account>/<mcp-name>:provider:<provider-name>`

Configure the _google.subject_ attribute as **assertion.sub**

Under _Attribute Conditions_ select **Add Condition**.

<!-- vale gitlab.Uppercase = NO -->
<!-- ignore CEL -->
To authenticate any managed control plane in your organization, in the _Conditional CEL_ input box put
<!-- vale gitlab.Uppercase = YES -->

{{<editCode >}}
```console
google.subject.contains("mcp:$@ORGANIZATION_NAME$@")
```
{{< /editCode >}}

{{< hint "warning" >}}
Not providing a CEL condition allows any Upbound managed control plane to access your GCP account if they know the project ID and service account name.
{{< /hint >}}

Select **Save**.

{{<img src="images/gcp-idp-attributes.png" alt="GCP configure provider attributes configuration" align="center" unBlur="true" size="small" >}}

### Create a GCP Service Account

GCP requires Upbound to use a [Service Account](https://cloud.google.com/iam/docs/service-account-overview). The required GCP _roles_ of the service account depend on the services managed by your control plane.

1. Open the **[GCP IAM Admin console](https://console.cloud.google.com/iam-admin/iam)**.
2. Select **[Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts)**.
3. From the top of the page, select **Create Service Account**.

##### Service account details
<!-- vale Google.WordList = NO -->
<!-- ignore account name -->
<!-- vale Google.FirstPerson = NO -->
Under _Service account details_ enter
_Service account name_: **upbound-service-account**
_Service account ID_: **upbound-service-account-id**
_Description_: **Upbound managed control planes service account**
<!-- vale Google.WordList = YES -->
<!-- vale Google.FirstPerson = YES -->

Select **Create and Continue**.

{{<img src="images/gcp-sa.png" alt="GCP service account creation screen" align="center" size="small" unBlur="true" >}}

##### Grant this service account access to project
For the _CloudSQL as a service_ configuration the service account requires the roles:
**Cloud SQL Admin**
**Workload Identity User**

Select **Done**.

{{<img src="images/gcp-role-add.png" alt="GCP service project access screen" unBlur="true" size="small" align="center">}}

##### Record the service account email address

At the list of service accounts copy the service account **email**.
Upbound requires this to authenticate your managed control plane.

{{<img src="images/gcp-sa-email.png" alt="list of GCP service accounts" unBlur="true" >}}

### Add the service account to the identity pool

Add the service account to the Workload Identity Federation pool to authenticate to Upbound with OIDC.
1. Return to the **[Workload Identity Federation](https://console.cloud.google.com/iam-admin/workload-identity-pools)** page and select the [**upbound-oidc-pool**](https://console.cloud.google.com/iam-admin/workload-identity-pools/pool/upbound-oidc-pool).
2. Near the top of the page select **Grant Access**.
3. Select the new service account, **upbound-service-account**.
4. Under _Select principals_ use **All identities in the pool**.
{{<img src="images/gcp-add-to-pool.png" alt="Pool select service account screen" align="center" size="small" unBlur="true">}}
Select **Save**.
In the _Configure your application_ window, select **Dismiss**.

<!-- vale Microsoft.Terms = NO -->
<!-- vale Google.Headings = NO -->
<!-- allow "Cloud". It matches the label on the page -->

<!-- vale Microsoft.HeadingAcronyms = NO -->
<!-- allow "SQL" -->
### Enable the Cloud SQL Admin GCP API
GCP requires explicitly enabling the Cloud SQL Admin API.


Go to the [Cloud SQL Admin API](https://console.cloud.google.com/apis/library/sqladmin.googleapis.com) page in the GCP console.


Select **Enable**.

{{<img src="images/enable-cloud-sql-api.png" alt="Enable the Cloud SQL Admin API in the GCP console" unBlur="true" size="small" >}}
<!-- vale Google.Headings = YES -->
<!-- vale Microsoft.Terms = YES -->

### Finish configuring the Upbound identity provider

Back in Upbound, finish configuring the identity provider.

In the _Identifier of GCP project_ field enter your **[GCP project ID](https://support.google.com/googleapi/answer/7014113)**.

For the _Name of federated identity provider_, edit your _GCP Project Number_ and enter:
{{< editCode >}}
```yaml
projects/$@<GCP Project Number>$@/locations/global/workloadIdentityPools/upbound-oidc-pool/providers/upbound-oidc-provider
```
{{< /editCode >}}

<br />

{{<hint "note" >}}
The identity provider format is:
```
projects/<GCP_PROJECT_NUMBER>/locations/global/workloadIdentityPools/<OIDC_POOL_NAME>/providers/<OIDC_POOL_PROVIDER_NAME>
```
{{< /hint >}}

<br />

In the _Identifier of GCP Project_ enter the Project ID.

In the _Name of federated identity provider_ field.

In the _Attached service account email address_ field enter the **service account email**.

{{<img src="images/gcp-oidc-finalize.png" alt="Upbound configuration to connect to GCP with OIDC" size="small" >}}

<!-- vale Upbound.Ampersand = NO -->
<!-- vale write-good.TooWordy = NO -->

Select **Finalize & Launch Control Plane**.

<!-- vale Upbound.Ampersand = YES -->
<!-- vale write-good.TooWordy = YES -->

{{< /tab >}}

{{< /tabs >}}

## Create your first resource

<div style="position: relative; padding-bottom: calc(75.92682926829268% + 42px); height: 0;"><iframe src="https://app.supademo.com/embed/VD6GeoOLStYRevpYlb28z" allow="clipboard-write" frameborder="0" webkitallowfullscreen="true" mozallowfullscreen="true" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

<!-- vale Google.FirstPerson = NO -->
<!-- allow "my" -->

Use the suggested values below for your cluster information:

- _name_: **my-app-cluster**
- _namespace_: **default**
- _id_: **my-app-cluster**
- _count_: **1**
- _size_: **small**
<!-- vale Google.FirstPerson = YES -->

Congratulations, you created your first resources with your MCP.

## Next steps

To learn more about the core concepts of Upbound, read the [concepts]({{<ref "concepts">}}) documentation. To learn how to begin building your own platform on Upbound, read the [Crossplane Architecture Framework]({{<ref "xp-arch-framework/_index.md">}}).
