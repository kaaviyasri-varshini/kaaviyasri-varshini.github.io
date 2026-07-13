Title: What's That Weird "CNAME" Thing in Your Domain Settings?
Date: 2026-07-13
Category: Developer Tools
Tags: dns, cname, domain, web, explainer
Slug: what-is-a-cname-record

## The Five-Minute Explainer for Anyone Who's Ever Copy-Pasted DNS Settings Without Knowing Why

Ever bought a domain, tried to connect it to some service like Shopify or Squarespace, and hit a screen asking you to add a "CNAME record"? You stare at it, shrug, copy-paste what they tell you, and move on — no idea what you just did.

Totally normal. Let's actually figure out what a CNAME is, in plain English.

## Think of It Like a Nickname

Imagine your friend Alex goes by "Al" to some people. If you ask "where's Al?" someone points you toward Alex, and *then* you find out where Alex actually is.

A **CNAME** (short for "Canonical Name") works the same way for websites. It's a DNS record that says: "Hey, this domain name isn't the real destination — go check *that* other domain name instead."

So instead of pointing straight to an IP address (the website's actual "location" on the internet), a CNAME points to *another domain name*, which then points to the real location.

## A Quick Example

Let's say you own `example.com`, and you want `www.example.com` to work too. You could set up:

```
www.example.com  →  CNAME  →  example.com
```

Now, whenever someone visits `www.example.com`, their browser says "oh, that's just an alias — let me go look up `example.com` instead," and it finds the site from there.

## Why Would Anyone Bother With This?

A few everyday reasons:

**Connecting a custom domain to a service.** Using Shopify, Squarespace, or a similar platform? They'll often ask you to point a subdomain (like `shop.yourbrand.com`) to *their* servers using a CNAME. That's how your fancy custom domain ends up showing their platform's content.

**Making future changes easier.** If the underlying server address ever changes, you only have to update it in one place — not on every single subdomain pointing to it.

**Keeping things organized.** Subdomains like `blog.yoursite.com` or `shop.yoursite.com` can each have their own CNAME, pointing wherever they need to.

## A Couple of Quirks Worth Knowing

Nothing too technical here, just two good-to-knows:

**CNAMEs can't share space.** If a domain name has a CNAME record, it can't have any other records (like email settings) attached to that same name. It has to be all-alias, nothing else.

**You can't put a CNAME on your "root" domain.** That means something like `example.com` itself (no `www`, no subdomain) usually can't use a CNAME — only subdomains can. If you need something similar at the root, providers often offer a workaround called an "ALIAS" or "ANAME" record.

## The Takeaway

A CNAME is really just a signpost that says "the real answer is over there." It lets one domain name borrow the identity of another, which makes connecting services, managing subdomains, and future-proofing your website way simpler than it sounds.

Next time a setup guide tells you to "add a CNAME record," you'll know exactly what it's asking — and why.
