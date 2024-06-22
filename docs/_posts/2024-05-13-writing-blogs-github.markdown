---
layout: post
title:  "Web-log (a.k.a blog) using GitHub pages"
date:   2024-05-13 14:00:00 +0800
categories: top scratch
mermaid: true
author: Isuru
---

# Writing blogs using GitHub pages

GitHub-pages is a way of hosting [Jekyll](https://jekyllrb.com/) based static sites.
I decided to use it to host my tech blog. After all we are all living in the age of **everything as code**. 

So why not **blog as code**. 

This post is a **TL;DR** and a compilation of sources for more details.

# Specification for a blog solution

1. Should have ability to run locally and test before publishing (offline support)
2. Version control
2. Use markdown as the formatter for writing
3. Have good templating support

# Solution 1 - Self Hosting

[Jekyll](https://jekyllrb.com/) is site generator. You can write content for your site using markdown format. 
There are many [themes](https://jekyllrb.com/docs/themes/) available as well, to bootstrap your site code. 
I used to host my blog by running jekyll server on [Amazon EC2](https://aws.amazon.com/pm/ec2/) instance and using
[AWS Route 53](https://aws.amazon.com/route53/) Authoritative DNS server to route traffic to the [public IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#concepts-public-addresses) of my EC2 instance. 
To enable TLS I used [Let's Encrypt](https://letsencrypt.org/) to get a server certificate which is valid for 3 months.
And use the [certbot](https://certbot.eff.org/) to renew my server certificate automatically. 

That's a lot of work, also recurring cost for AWS infrastructure.

# Solution 2 - Using GitHub Pages

You have to create a new github repository with name <your-github-account-id>.github.io. 

For an example, my GitHub username is busy-spin. And the repository I use to host my blog is [busy-spin.github.io](https://github.com/busy-spin/busy-spin.github.io).
Publishing content to your website is as simple as pushing content to GitHub repository.

I can run the blog site locally as both approaches use Jekyll site generator. 

# Testing Site Locally

Bellow is a complete guide on how to test your site before publishing.

[Guide - Testing your side locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

### Important commands are 

To install the required Ruby packages
```shell
bundle install
```

To run the jekyll server locally
```shell
bundle exec jekyll serve
```

Jekyll process has a file watcher so once you update the content it will auto compile it to html code. 


## Extras

### Adding mermaid flow chart support for GitHub pages.

[mermaid.js](https://mermaid.live/) chart support is available by default in GitHub wiki, GitHub projects. 
But enabling [mermaid.js](https://mermaid.live/) for GitHub pages is not supported out of box and also not trivial to enable. 
I found this neat trick from fellow GitHub blogger [jackgrubber](https://github.com/JackGruber/jackgruber.github.io) - [Embed-Mermaid-in-Jekyll-without-plugin](https://jackgruber.github.io/2021-05-09-Embed-Mermaid-in-Jekyll-without-plugin/).

<pre>
```mermaid
flowchart TD
    A[Learn difficult concepts] -->|Create Experiment| B{Experiment}
    B --> | Too hard to understand | C[Read more and write about it]
    B --> | Got a good grip of concept | D[Write about it]
    C --> | Experiment Again | B
    D --> | Level up again | A
```  
</pre> 

```mermaid
flowchart TD
    A[Learn difficult concepts] -->|Create Experiment| B{Experiment}
    B --> | Too hard to understand | C[Read more and write about it]
    B --> | Got a good grip of concept | D[Write about it]
    C --> | Experiment Again | B
    D --> | Level up again | A
```  


# Other Helpful References

[Jekyll Quick Start](https://jekyllrb.com/docs/)

[GitHub Pages - Quick Start](https://docs.github.com/en/pages/quickstart)

[Writing on GitHub](https://docs.github.com/en/get-started/writing-on-github) - A complete guide on markdown

[GitHub - Basic Writing And Formatting Syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
