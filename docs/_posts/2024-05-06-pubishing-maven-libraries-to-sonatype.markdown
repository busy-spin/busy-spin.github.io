---
layout: post
title:  "Publishing java libraries to Sonatype"
date:   2024-05-05 18:00:00 +0800
categories: top scratch
author: Isuru
---

# Step-by-step guide to publish your library from GitHub to Sonatype

## TL;DR

As a developer I use Github to share my code with others. Being a java developer, I also see the advantage 
of using java libraries as a form of sharing our work. Once java library is developed with useful release features, 
it can be deployed to [Sonatype](https://central.sonatype.com/) maven central repository. 
Github offers the [GitHub Actions](https://docs.github.com/en/actions) where we can seamlessly automate from code commit to library publish. 

## How

1. Create sonatype account, get token to publish maven artefacts
2. Claim the namespace so you can publish under maven `<groupId>`
3. Create gpg keys, include gpg private key in your build and add the public key in to key server, this is to perform integrity checks for libraries you publish 
4. Set up secrets for your repository including sonatype username, token, gpg private key, gpg passphrase
5. Set up GitHub action workflow so you can publish libraries from GitHub to Sonatype


## Why

### Claiming you namespace

All my libraries go under `<groupId>io.github.busy-spin</groupId>` I should claim this namespace in Sonatype. 
Pretty much similar to how you get a username or blog-space.

### GPG private key and public key

Standard why to perform a integrity check for your libraries is to generate GPG key pair. 
Keep the private key with you (In this case keep it in GitHub Secrets).
And publish the public key to gpg key server. When you publish the artefacts, the artefacts will be signed using your private key. 
And Sonatype server will verify using the published public key.

### Why using GitHub Actions / GitHub Secrets

As your library start getting traction, more and more people would want to collaborate with you. 
It's easy to centralize the publishing instead of all contributors sharing GPG keys and other secrets and 
settings up the deployment related settings in their individual systems. 

## Steps

1. Create sonatype account and get token

Use following link to create account in Sonatype and create user token.

[Generate Sonatype User Token](https://central.sonatype.com/account)

2. Claim your namespace

Use the following link to claim the namespace which is your maven `<groupId>`

[Claim Namespace](https://central.sonatype.com/publishing/namespaces)

3. Generate GPG key

Use following guide on generating gpg-key pair and publishing the public key to gpg key server

[Generate GPG Keys](https://central.sonatype.org/publish/requirements/gpg/)

Use following command to copy your GPG private key, replace `id` with your GPG key pair id.
GPG private key and GPG passphrase will be later uploaded in to GitHub secrets.

```shell
gpg --armor --export-secret-key <id>
```

4. Setting up secrets

Use following guide on more details on settings up secrets.

[Adding Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)

To support the GitHub Actions that I will later set up. I need to configure four secrets



## References

[Publishing Deployment](https://central.sonatype.com/publishing/deployments)
