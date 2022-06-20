---
title: New Website!
date: 2022-06-19 12:00:00 -500
categories: [misc]
tags: [website]
---

# Introduction

After a number of years hosting my personal site on WordPress I've decided to move off the platform. In this blog post I'll describe some of the reasons why for those who are curious. If you're not curious, the new domain is here: [jack.kawell.us](https://jack.kawell.us)

Any new posts will be here on the new site and all existing posts have been copied here already. I've also posted notes on each of the old posts redirecting you to the new site. Not sure how long I'll keep the old WordPress site up before I just have DNS redirect all traffic from [jack-kawell.com](https://jack-kawell.com) to [jack.kawell.us](https://jack.kawell.us).

# Details

## Why change?

I originally started this site on WordPress just because I wanted an easy place to host my personal site to keep a resume and post random tutorials. It was cheap and easy to get up and running here and for the most part I've really liked it.

Recently, however, I've been diving deep into FOSS (free open-source software) tools and self-hosting. In order to host this WordPress site I pay ~100USD/year so that I can host it at a custom domain. This isn't terrible for someone who doesn't have any technical prowess, but since I literally run software systems for a living it seems ridiculous to me to pay someone else to run things when I can do it for (almost) free and have a ton of fun learning some new tools.

I've also been frustrated at the "freemium" model that WordPress runs on. They lure you in with many promises of how amazing your site can be but so much of that value is locked behind premium subscriptions and plugins. I just want a simple site where I can publish my content and I don't want to have to pay for things as simple as dark mode.

## What am I doing now?

I've decided to move to using [Jekyll](https://jekyllrb.com/) which is a dead simple static site generator. It allows you to use Markdown (which I love!) to write your site and has a huge amount of templates to choose from to hone in one the style and theme you want. It also is completely free and open-source and is the technology powering GitHub Pages so you can host it through GitHub for free and rely on GitHub's uptime guarantees.

I thought about self-hosting my site but since I spend a lot of time tinkering with my homelab I didn't want to be dependent on having 100% uptime for the thousands (!!!) of monthly visitors who use my tutorials. So I'm going the route of hosting the site on GitHub and publishing it using GitHub pages. If you're curious about the code you can find it here: [github.com/jgkawell/jgkawell.github.io](https://github.com/jgkawell/jgkawell.github.io)

I decided to use the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy/) as my Jekyll theme since it's super clean, and has all the things I want for my blog: dark mode, social links, archive timeline, tags and categories, etc. Further, it has built in commenting using [giscus](https://giscus.app/) which is an awesome plugin to have commenting on a blog powered through GitHub Discussions.

The final awesome piece is that all the configuration of the site is simply text files that I can edit just like all the other code I work with day to day. I can even check in changes to the git repo and it will automatically build and deploy it using GitHub Actions! CI/CD for the win!

One last implementation detail. I used to use Google Domains as my registrar and DNS provider. I've since moved to Cloudflare and my costs were cut by 2/3! I'm a huge fan of Cloudflare after using them for the past month and now have multiple domains registered through them. Their free tier is awesome and they don't charge registrar fees on top of the mandatory ones (like ICANN).

# Conclusion

As you can tell, I'm loving this new set up and hope to be posting new content soon. If you have a site hosting tool you really like I'd love to hear about it. Bonus points if it's free and open-source. These types of tools have just been getting better over time and it makes me surprised I went so long paying for WordPress before changing.
