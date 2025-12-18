+++
title = 'Making a Self Hosted Blog'
date = 2025-12-17
draft = false
+++

 ## Project Overview

I've been wanting to make my own site for a while. For a few different reasons (mainly cost, but also for the learning opportunity), I leaned towards self hosting it. Since I already have a server running, I figured it would be an easy process. Other than a few headaches with DNS (because it's always DNS), the process was relatively straight forward. With this workflow, I'm able to write pages and posts locally on any computer, push the changes to my [github repo](https://github.com/tylermc94/dev-blog), and the updated repo is automatically pulled to the server, turned into html by Hugo, and then moved to the nginx container to run the site.

## What I Learned

- Setting up a Cloudflare Tunnel and connecting it to my home server
- Using Hugo to create an html site from simple markdown documents
- Creating cron jobs in Unraid to run custom scripts

## How I Did It

My main goal in this whole project was to do it as securely as possible. Other than random temporary port forwarding for game servers I've used over the years, I've never really exposed anything on my network to the public internet. The thought of doing it was a little scary, so I spent a good amount of time looking into the best ways to do it. I ended up going with Cloudflare Tunnel.

All it took was installing the [Cloudflare Tunnel Client](https://github.com/cloudflare/cloudflared) on my Unraid server. There is a container in Community Applications, cloudflaredtunnel by Cloudflare, that makes it super easy. While setting up that container, I needed a tunnel ID. I bought my domain on Cloudflare, so I already had an account. In the Zero Trust section, I was able to set it up as a Connector. You get the tunnel ID, plug it into the docker container, and I had a secure connection from outside to inside your network with no port forwarding or DDNS.

Once the tunnel is active, you just have to configure the DNS. This was where I ran into trouble. I had assumed that the DNS should be configured on the domain itself, since it was covered in warnings that if I didn't set up DNS, horrible things would happen.This obviously didn't work, and I spent way too long digging through the domain and DNS settings trying to figure out how to link it to the tunnel. As I eventually discovered, I was looking at the wrong side of it. The Tunnel configuration page has a section for Published Application Routes, where you can configure a DNS route to go from a public domain (blog.elevation1505.dev) to a private IP. Last was just setting a policy on that application to bypass authentication, so the site doesnt require a login.

The last thing I needed to do was make a way to edit the site. I installed [Hugo](https://github.com/gohugoio/hugo) on my server and set up a really simple script to pull from my github repo every 10 minutes using the User Scripts plugin in Unraid. 

```
#!/bin/bash

# Navigate to your blog directory
cd /mnt/user/appdata/blog

# Pull latest changes from GitHub
git pull origin main

# Rebuild the Hugo site
hugo --destination /mnt/user/appdata/nginx/html
```

I set that to run every 10 minutes in User Scripts and that's it! I update the markdown files on my laptop, push to github, the server pulls from github, then runs the command for Hugo to rebuild the site and put it in the nginx directory.

## What's Next

- I want to change the cronjob on the server to a webhook to get rid of the potentially up to 10 minute delay between pushing the updated site and the site updating.
- I have to make it prettier. Right now its just a black header with my name and then 2 posts and an about me page. Hugo has tons of themes to pick from, and a huge community making their own themes. 
- Before I even wrote this article I already had a request to turn on comments. (thanks Kathy) I want to add comments, likes, and just generally anything that makes the site more friendly.