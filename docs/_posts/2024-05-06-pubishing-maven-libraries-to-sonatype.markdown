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

1. Create sonatype account, get credentials to publish maven artefacts
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

## References
[Github Actions](https://docs.github.com/en/actions/publishing-packages/publishing-java-packages-with-maven)
[Generate Sonatype User Token](https://central.sonatype.com/account)
[Claim Namespace](https://central.sonatype.com/publishing/namespaces)
[Generate GPG Keys](https://central.sonatype.org/publish/requirements/gpg/)
[Adding Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
[Publishing Deployment](https://central.sonatype.com/publishing/deployments)
