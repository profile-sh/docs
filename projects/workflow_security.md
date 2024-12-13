---
layout: page
title: Workflow Security
description: Security for github workflows
giscus_comments: true
author: {{site.author}}
date: 12-12-2024
---

In CI/CD cycles, GitHub actions and workflows very likely will need access to resources and services on GitHub or outside GitHub. Also, 
the outside services may need access to our resources on GitHub. It is very important to understand how to securely stitch 
together services and resources (public or private) in the CI/CD workflows.

## 1. GitHub Token

Visit [GitHub docs] (https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication) 
for further reading.

Main points:
- GitHub token, as the name suggests, is a *token* provided by GitHub.
-  used to manage access permissions in GitHub actions and workflows.
- it is generated at the start of each job by a GitHub app installed in every actions-enabled repository by default.
- the token is specific to a repository and a workflow run, it expires after the job finishes or after 24hrs max.
- use it in a workflow: secrets.GITHUB_TOKEN or github.token.
- When used in a reusable workflow, the token  corresponds to the caller repo.
- the default access permissions of the token can be configured in actions settings for each repo or orgnaization.
- the default permissions may be *restricted* or *permissive*: [default access](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)
- the defaults can be overridden explicitly in a workflow at the top level or the job level using the *permissions* key.

## 2. Personal Access Token (PAT)

- PAT is configured at GitHub user profile level using *settings*,  access to repos is configurable.
- PAT is used for remote access or to provide access to one or more repositories that may or may not be the caller-workflow-repo.
- (PAT docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## 3. GitHUb OpenID Connect for AWS

This is one of the methods to define access interface between AWS and GitHub. This is needed to provision resources on AWS.

The aws official action aws-actions/configure-aws-credentials supports five methods for fetching credentials from AWS, 
OpenID Connect is the recommended one as it gets short-lived credentials, enough for actions and workflows. Visit [configure-aws-credentials action docs](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#oidc)
and (GitHub docs](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#adding-the-identity-provider-to-aws) 
for further reading. The AWS provided action also allows to limit access using inline policies as inputs.

To configure GitHub as OPENID Connect indetity provider, visit IAM console on aws:

- Add GitHub as identity provider:
  - provider: token.actions.githubusercontent.com
  - audience: sts.amazonaws.com
2.  Add a role, for a *web identity* and your GitHub organization (repo name or branch are optional) and
    select or define a policy to attach with the role, the end result may look like:

Example trust policy for the IAM role to allow access from all repos under the org YOUR_ORG_NAME:

{% raw %} 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::************:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG_NAME/*"
                }
            }
        }
    ]
}
```
{% endraw %}

