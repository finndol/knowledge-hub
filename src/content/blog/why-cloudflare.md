---
title: "Why I chose Cloudflare"
description: "What edge platforms get right, and why I'm betting on Cloudflare for this project."
pubDate: 2026-05-25
tags: ["cloudflare", "infrastructure"]
---

## The traditional web is regional

For most of the web's history, your app lived in one data center. Maybe AWS us-east-1. Maybe a server in your closet. Every request from every user, anywhere in the world, traveled to that one place. People in Sydney loaded sites slower than people in San Jose, not because of their internet, but because of physics.

## Edge computing flips the model

Cloudflare runs code in 300+ data centers worldwide. When a user in Tokyo visits your site, your code runs in Tokyo. Static assets are served from Tokyo. Even server-rendered pages execute in Tokyo. The round trip never crosses an ocean.

## Why I'm building here

I want to internalize this model — not by reading about it, but by building something on it. Astro plus Cloudflare gives me a real environment where every feature I add teaches me a piece of how edge platforms actually work.