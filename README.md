# Google Cloud - SendGrid Email Sending

This is an example of how to send emails in [Google Cloud](https://cloud.google.com/) via [SendGrid](https://sendgrid.com/).

## System Overview

## Architecture Diagram

![system architecture diagram](./images/system-architecture-diagram.png)

## Components

### [sendgrid Cloud Function](./cloud-functions/sendgrid)

A [Pub/Sub triggered Cloud Function](https://cloud.google.com/functions/docs/calling/pubsub) that sends emails using [SendGrid](https://sendgrid.com/).

It expects a message with the following structure:

```json
{
  "to": "recipient@example.com",
  "subject": "My email subject",
  "body": "<h1>The email body can be html</h1>"
}
```

## Deployment

The system is deployed using [`terraform`](https://www.terraform.io/), running in [Cloud Build](https://cloud.google.com/build/docs/overview).

### Pre-requisites

1. [Create an API key](https://app.sendgrid.com/guide/integrate/langs/nodejs) in SendGrid.
1. [Set up domain authentication](https://docs.sendgrid.com/ui/account-and-settings/how-to-set-up-domain-authentication) in SendGrid.
1. Create a Google Cloud [Organization](https://cloud.google.com/resource-manager/docs/creating-managing-organization).
1. Install [`terraform`](https://developer.hashicorp.com/terraform/downloads).
1. Install the [`gcloud` CLI](https://cloud.google.com/sdk/docs/install).

### Bootstrap

This is the process that creates the Google Cloud [Project](https://cloud.google.com/resource-manager/docs/creating-managing-projects), [enables the required APIs](https://cloud.google.com/apis/docs/getting-started), and grants the necessary permissions to the [Service Accounts](https://cloud.google.com/iam/docs/service-accounts), including the ones required for the [Cloud Build Service Account](https://cloud.google.com/build/docs/cloud-build-service-account) to deploy the system.

1. Run `gcloud auth login`
1. Run `gcloud auth application-default login`.
1. `cd` into the [deployment/google-cloud/terraform/bootstrap](./deployment/google-cloud/terraform/bootstrap) folder.
1. Comment out the entire contents of the [deployment/google-cloud/terraform/bootstrap/backend.tf](deployment/google-cloud/terraform/bootstrap/backend.tf) file.
1. Create a [`terraform.tfvars`](https://developer.hashicorp.com/terraform/language/values/variables#variable-definitions-tfvars-files) file and add your variables' values. Leave the `sourcerepo_name` empty for now.
1. Run `terraform init`.
1. Run `terraform apply -target=module.project`.
1. Uncomment the [deployment/google-cloud/terraform/bootstrap/backend.tf](deployment/google-cloud/terraform/bootstrap/backend.tf) file's contents and add the value of the `tfstate_bucket` output as the value of the `bucket` attribute.
1. Run `terraform init` and answer `yes`.
1. [Create a Cloud Source Repository](https://cloud.google.com/source-repositories/docs/creating-an-empty-repository#gcloud) in the project your just created. Optionally, [fork this repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo) and create a Cloud Source Repository by [mirroring your forked repo](https://cloud.google.com/source-repositories/docs/mirroring-a-github-repository). Update the `sourcerepo_name` variable with the repository name.
1. Run `terraform apply`.

### Deployment

This is a Cloud Build [build](https://cloud.google.com/build/docs/overview#how_builds_work) that actually deploys the system.

1. The pipeline can be triggered by either:
   - Push a commit to your Cloud Source Repository or to your Github fork.
   - Go to your project's [Cloud Build Dashboard](https://console.cloud.google.com/cloud-build/triggers) and manually run the `push-to-branch-deployment` [trigger](https://cloud.google.com/build/docs/triggers).
