---
title: "Fixing Broken Links in GitHub Pages — Beginner’s Guide to baseurl, relative_url, and Slugify"
date: 2025-11-17
categories: [GitHub Pages, DevOps & Automation]
tags: [GitHub-Pages, Jekyll, baseurl, relative_url, slugify]
---


## Fixing Broken Links in GitHub Pages

A guide to `baseurl`, `relative_url`, and `slugify` (with real examples).

> Optional: If you want to learn more about GitHub Pages migrations, check out my previous post: [Migrate a GitHub Pages Site Between Accounts — Step-by-Step Guide](https://mradelvand.github.io/aws/github%20pages/devops%20&%20automation/2025/11/17/migrate-github-page-site-between-accounts.html)


When creating two GitHub Pages project sites:
```bash
username.github.io/aws
username.github.io/soccer
```

a common problem appears — all the blog posts inside the AWS site open incorrect URLs.

Instead of:

> https://username.github.io/aws/aws-architecture/2025/10/28/post1.html


the URLs look like:

> https://username.github.io/aws-architecture/2025/10/28/post1.html


This means:

- Pages were broken  
- Categories were broken  
- Latest posts links were broken  

After investigating how Jekyll handles paths, here’s a simple explanation.

---

## Lesson 1 — A GitHub Pages Project Site Always Lives Inside a Folder

If your repository is named `aws`, the project site URL will always be:

> https://username.github.io/aws/


Every internal link must also start with `/aws/`, but Jekyll doesn’t automatically know this.  
You have to tell it where your site lives.

---

## Lesson 2 — `baseurl` Tells Jekyll “My Site Starts Inside a Folder”

Inside your `_config.yml`, add:

```bash
baseurl: "/aws"
```
This setting tells Jekyll:

Every URL should start with /aws

This is required for any GitHub Pages project site and fixes many link issues.

## Lesson 3 — relative_url Automatically Adds baseurl to Links

Before the fix, your archive or index page might have links like:
```bash
<a href="{{ post.url }}">{{ post.title }}</a>
```
post.url contains only the post path, for example:

```bash
/aws-architecture/2025/10/28/post1
```
But this creates a broken URL because the /aws/ prefix is missing.

By adding | relative_url:
```bash
<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
```

Jekyll automatically inserts the *baseurl* prefix:

This fixes all post links, archive links, and latest posts links.

## Lesson 4 — What is “Slugify” and Why It Matters

In your *archive.md* file, you might see:
```bash
<h2 id="{{ category[0] | slugify }}">{{ category[0] }}</h2>
```
**Slugify** is a Jekyll function that turns messy text into a clean, URL-friendly format for HTML IDs.

### Why this matters:

HTML IDs cannot contain spaces, capital letters, or special characters

Slugify ensures the category heading ID is safe for the browser

> Important: Slugify is applied only to the HTML ID, not the folder name.
> Folder names are still based on the category name you set in your post front matter.

## Lesson 6 — Summary

- baseurl tells Jekyll where your site lives

- relative_url automatically adds the baseurl to all links

- slugify cleans category names for safe HTML IDs

- Old posts can still work after adding baseurl + relative_url even if category names have spaces

These three concepts prevent broken links on GitHub Pages and make your site more consistent and beginner-friendly.
