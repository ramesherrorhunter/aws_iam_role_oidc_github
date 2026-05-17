# GitHub Actions OIDC with AWS IAM (DevOps Guide)

## Overview

This guide explains how to integrate GitHub Actions with AWS using OpenID Connect (OIDC) for secure, secretless authentication. This approach removes the need for long-lived AWS access keys in CI/CD pipelines.

I have been using this setup in production DevOps environments for years and recently shared it with the community to help others adopt a more secure approach.

---

## Why OIDC Instead of Access Keys?

Traditional CI/CD setups use AWS access keys stored in GitHub Secrets. This introduces risks:

* ❌ Long-lived credentials
* ❌ Secret leakage risk
* ❌ Manual rotation overhead
* ❌ Hard to audit and control

With OIDC:

* ✅ Short-lived credentials
* ✅ No secrets stored in GitHub
* ✅ Fine-grained IAM trust control
* ✅ Improved security posture

---

## How It Works

GitHub Actions uses OIDC tokens issued by:

```
token.actions.githubusercontent.com
```

Flow:

1. GitHub Actions workflow starts
2. OIDC token is generated automatically
3. AWS validates token via IAM trust policy
4. AWS STS assumes IAM role
5. Temporary credentials are issued
6. Deployment runs securely

---

## GitHub Actions Configuration

You must enable OIDC permissions in workflow:

```yaml
permissions:
  id-token: write
  contents: read
```

---

## AWS IAM Role Trust Policy

Example trust policy allowing GitHub repository access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:ramesherrorhunter/*"
        }
      }
    }
  ]
}
```

---

## Example GitHub Actions Workflow

```yaml
name: AWS OIDC Deployment

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/aws-iam-role-oidc-github
          aws-region: ap-south-1

      - name: Verify AWS Identity
        run: aws sts get-caller-identity
```

---

## Verification Output

Expected result:

```json
{
  "Account": "<ACCOUNT_ID>",
  "Arn": "arn:aws:sts::<ACCOUNT_ID>:assumed-role/aws-iam-role-oidc-github/..."
}
```

---

## Repository Reference

Full working example:

[https://github.com/ramesherrorhunter/aws_iam_role_oidc_github](https://github.com/ramesherrorhunter/aws_iam_role_oidc_github)

---

## Final Thoughts

This setup is one of the most impactful DevOps security improvements you can implement:

* Removes secrets from CI/CD
* Reduces attack surface
* Scales cleanly across repositories
* Aligns with cloud-native security best practices

---

## Tags

#DevOps #AWS #GitHubActions #OIDC #CloudSecurity #CICD #IAM #Kubernetes #PlatformEngineering #CloudNat
