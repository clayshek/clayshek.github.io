---
layout: post
title: Cloudflare DNS Terraform Cloud
tags: terraform cloudflare dns github github-actions terraform-cloud
excerpt_separator: <!--more-->
---

## **What**
* A GitHub hosted Terraform config for Cloudflare DNS, with [Terraform Cloud](https://cloud.hashicorp.com/products/terraform) integration via [GitHub Actions](https://github.com/features/actions).

## **Why**
* Codify all the things / [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code)
* Remote state management in Terraform Cloud
<!--more-->
* No more manual DNS udpates

## **How**

High-level summary - the Terraform config containing DNS zone record definitions will live in a GitHub repo. On pushes to the main branch, a GitHub Action, using the [hashicorp/setup-terraform](https://github.com/hashicorp/setup-terraform) action, will run the Terraform config in an API/CLI driven workspace on Terraform Cloud, and update Cloudflare. 

**Prerequisites:**
* A [Cloudflare](https://www.cloudflare.com/plans/#overview) hosted DNS zone (free plan is fine)
* [Terraform Cloud](https://cloud.hashicorp.com/products/terraform) Organization account (free plan is fine)
* GitHub account

**Steps:**
* Create new GitHub repo, fork or use as example [clayshek/cloudflare-dns-terraform-cloud](https://github.com/clayshek/cloudflare-dns-terraform-cloud). This uses the [Cloudflare provider](https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs)
* Create new Terraform Cloud Workspace, of type API-driven workflow
* In Terraform Cloud, [create a new Token](https://app.terraform.io/app/settings/tokens) for GitHub Actions to use
* Add a new Actions [Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to your GitHub repo, named TF_API_TOKEN, for the token created in prior step
* In Cloudflare, create an API token, using "Edit zone DNS" token template, to enable updating the zone in scope
* Add a new Actions Secret to your GitHub repo, named CLOUDFLARE_API_TOKEN for the token created in prior step
* If all is setup correctly, updates to the repo's main branch should start an Action initiated run in Terraform Cloud

### Note:
An alternative approach to the above would be a [VCS integrated TF Cloud Workspace](https://www.terraform.io/cloud-docs/vcs/github-app). Under this model, the GitHub Actions component would not be required. TF Cloud would monitor the repo for changes, and run terraform plan/apply directly in the TF Cloud Workspace. This would require some minor TF config updates, and adding an environment variable to the TF Cloud Workspace for CLOUDFLARE_API_TOKEN instead of as a GitHub secret.
Personally prefer the API driven workspace, enabling all config, including API token secret, to live in GitHub.

### References: 
* https://learn.hashicorp.com/tutorials/terraform/github-actions
* https://learn.hashicorp.com/collections/terraform/cloud-get-started
* https://registry.terraform.io/providers/cloudflare/cloudflare/latest/docs
* https://blog.cloudflare.com/getting-started-with-terraform-and-cloudflare-part-1/ 
* https://brendanthompson.com/posts/2021/09/triggering-terraform-cloud-runs-from-github