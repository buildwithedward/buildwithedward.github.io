---
layout: single
title: "From GitHub Pages to a Real Domain - How I Finally Claimed edwardpraveen.com"
date: 2026-05-24
categories: [devops, infrastructure]
tags: [devops, github-pages, dns, namecheap, aws, blogging, infrastructure]
excerpt: "A step-by-step account of moving my technical blog off a github.io subdomain, the architecture decisions I weighed, and exactly what I clicked (and skipped) along the way."
header:
  teaser: /assets/img/banner-custom-domain.png
---

I've been meaning to do this for a long time. My blog has been sitting at `buildwithedward.github.io` for years - functional, free, and completely forgettable as a URL. If I'm going to write seriously about production AI systems and infra, the least I can do is own my own domain.

This week I finally did it. Here's exactly how it went.

---

## The itch: why a custom domain now?

The github.io URL always felt temporary. It's fine when you're experimenting, but I'm now writing about real production topics - RAG pipelines, LLM inference, AWS architecture, client RFP approaches. A personal domain signals that this is a real, long-term effort. It's also just easier to share verbally: *"edwardpraveen.com"* versus *"buildwithedward dot github dot io."*

---

## The two options I considered

Before buying anything, I thought through the two realistic paths.

### Option 1 - Move hosting to AWS (S3 + CloudFront + Route 53)

**Advantages:**
- Full control over CDN behaviour, caching rules, and redirects
- Custom edge logic if needed in the future
- Everything under one AWS account alongside other infra

**Considerations:**
- ~$15–20/yr total cost (Route 53 hosted zone + S3 + CloudFront)
- Requires a CI/CD pipeline (GitHub Actions → S3 sync + CloudFront invalidation)
- More moving parts to maintain for what is ultimately a static site
- Overkill for a Jekyll blog that rebuilds on every push

### Option 2 - Keep GitHub Pages, add a custom domain ✓ *Chosen*

**Advantages:**
- GitHub handles SSL automatically via Let's Encrypt - no cost, no renewal headache
- Zero infrastructure to manage
- The old `buildwithedward.github.io` URL auto-redirects to the new domain
- Domain cost only (~$10/yr)

**Considerations:**
- No server-side logic or edge functions
- GitHub Pages uptime is tied to GitHub's infrastructure (historically excellent)

**Why I chose this:** AWS makes sense when you need custom server-side behaviour, granular CDN control, or want everything under one account for compliance reasons. For a static Jekyll blog, GitHub Pages does everything needed. Right tool for the job - no point adding complexity to a solved problem.

---

## Buying the domain: what I skipped at Namecheap checkout

I went with Namecheap and grabbed `edwardpraveen.com`. The checkout page, as always, tries to sell you a bundle of add-ons. Here's my honest take on each:

| Add-on | Cost | Verdict |
|--------|------|---------|
| SSL Certificate | ~$15–50/yr | ❌ Skip - GitHub Pages gives free SSL via Let's Encrypt. ACM is also free if you move to AWS later. |
| Premium DNS | ~$5/yr | ❌ Skip - BasicDNS is perfectly adequate for a personal blog. Premium adds DDoS protection you don't need. |
| Sitelock Website Security | ~$20/yr | ❌ Skip - Sitelock scans PHP/WordPress sites for malware. A static Jekyll site has nothing to infect. |
| Business Email | ~$15/yr | ⚠️ Optional - worth it for `edward@edwardpraveen.com`, but Zoho Mail has a free tier for one custom address. |

Total checkout: ~$10 for the domain alone.

---

## The actual setup: what I changed

Once the domain was registered, the changes were split across two places.

### Namecheap - Advanced DNS

Namecheap pre-populates two default records out of the box:
- A CNAME pointing `www` to `parkingpage.namecheap.com`
- A URL Redirect Record on `@`

**Delete both of these first.** They will conflict with the GitHub records.

Then add the following:

**4 A records** (GitHub's servers):

| Type | Host | Value |
|------|------|-------|
| A Record | @ | 185.199.108.153 |
| A Record | @ | 185.199.109.153 |
| A Record | @ | 185.199.110.153 |
| A Record | @ | 185.199.111.153 |

**1 CNAME record** (www subdomain):

| Type | Host | Value |
|------|------|-------|
| CNAME Record | www | buildwithedward.github.io. |

> Note the trailing dot after `.github.io.` - include it exactly as shown.

### GitHub repository

1. Add a `CNAME` file in the root of your repo containing just your domain:
   ```
   edwardpraveen.com
   ```

2. Go to **Settings → Pages → Custom domain**, enter `edwardpraveen.com`, click Save.

3. Wait for GitHub's DNS check to pass (a few minutes), then tick **Enforce HTTPS**.

4. Update `_config.yml`:
   ```yaml
   url: "https://edwardpraveen.com"
   baseurl: ""
   ```

Commit and push. GitHub rebuilds the site and the SSL cert provisions automatically within ~30 minutes.

> The old `buildwithedward.github.io` URL automatically issues a **301 redirect** to `edwardpraveen.com` once the custom domain is set - no extra config needed.

### Verify everything works

Once DNS propagates (30 min to 48 hrs), check:

- `https://edwardpraveen.com` loads the blog
- `https://www.edwardpraveen.com` loads the blog
- `http://edwardpraveen.com` redirects to `https://`
- `buildwithedward.github.io` redirects to `edwardpraveen.com`
- Padlock/SSL shows in the browser

You can monitor DNS propagation in real time at [dnschecker.org](https://dnschecker.org).

---

## What's next for this blog

The domain was the activation energy I needed. Going forward, I'm planning to publish weekly - covering the things I'm actually working on:

**Infrastructure and DevOps** - real-world AWS architectures, ECS deployments, CI/CD pipelines, IaC patterns. The kind of end-to-end walkthroughs I wish existed when I was setting things up.

**AI/ML engineering** - LLM inference optimization, RAG pipeline design, multi-agent orchestration. Practical content grounded in production constraints, not toy examples.

**Architecture approaches from client RFPs** - I've been involved in scoping and proposing systems for enterprise clients. I'll share the architectural thinking behind those proposals, with synthetic data where needed, so the reasoning is transferable even when the specifics can't be.

---

If you've been putting off claiming your own domain for a blog or portfolio, the barrier is genuinely low - one afternoon, ~$10, and a handful of DNS records. The github.io redirect means you lose nothing from your existing audience. There's no good reason to wait.
