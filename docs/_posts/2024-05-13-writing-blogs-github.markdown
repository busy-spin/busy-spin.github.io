---
layout: post
title:  "Web-log (a.k.a blog) using GitHub pages"
date:   2024-05-13 14:00:00 +0800
categories: top scratch
author: Isuru
---

# Writing blogs using GitHub pages

As GitHub-pages is a way of hosting [Jekyll](https://jekyllrb.com/) based static sites, 
I decided to use it to host my tech blog. After all we are all living in everything as code age. 

So why not **blog as code**. 

This post is a **TL;DR** and a compilation of sources for more details.

# Specification for a blog solution

1. Should have ability to run locally and test before publishing
2. Use markdown as the formatter for writing
3. Have good templating support

# Solution 1 - Self Hosting

[Jekyll](https://jekyllrb.com/) is site generator. You can write content for your site using markdown format. 
There are many of [themes](https://jekyllrb.com/docs/themes/) available as well to bootstrap your site code. 
I use to host my blog by running jekyll server on Amazon EC2 instance and using 
AWS Route 53 DNS server to route traffic to the public IP of my EC2 instance. 
To enable HTTPS I used [Let's Encrypt](https://letsencrypt.org/) to get a server certificate which is valid for 3 months.
And use the [certbot](https://certbot.eff.org/) to renew my server certificate automatically. 

That's a lot of work, also recurring cost.

# Solution 2 - Using GitHub Pages

You create a new github repository with name <your-github-account-id>.github.io. 

For an example, my GitHub username is busy-spin. And the repository I use to host my blog is `busy-spin.github.io`.
Publishing content to your website is as simple as pushing content to GitHub repository.

I can still run the blog site locally as both approaches use Jekyll site generator. 

# Testing Site Locally

Following page from GitHub outlines how to test your site before publishing

# Other Helpful References

[Jekyll Quick Start](https://jekyllrb.com/docs/)

[GitHub Pages - Quick Start](https://docs.github.com/en/pages/quickstart)

[Writing on GitHub](https://docs.github.com/en/get-started/writing-on-github) - A complete guide on markdown

[GitHub - Basic Writing And Formatting Syntax](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
