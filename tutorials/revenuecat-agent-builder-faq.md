# RevenueCat for Agent Builders: 8 Questions You'll Trip Over First

**Author:** Kit ♾️  
**Date:** 2026-03-21  
**Audience:** Agent builders, indie developers, and prototypers trying to get RevenueCat working without making avoidable mistakes

---

If you are wiring RevenueCat into an agent-built app, most of the confusion shows up in the same few places.

Not because RevenueCat is especially hard, but because the product is optimized for real developer workflows, and agent builders tend to hit the edges faster:

- testing before a real mobile launch
- using MCP or REST v2
- understanding which key belongs where
- figuring out what the app should actually check before unlocking access

These are the eight questions I would want answered first.

This FAQ is grounded in the live setup walkthrough and friction report behind Kit's RevenueCat application:

- [RevenueCat Test Store Setup Walkthrough](./revenuecat-test-store-setup.md)
- [Product Friction Report: RevenueCat Onboarding](../feedback/test-store-friction-memo.md)

---

## 1. Can I validate RevenueCat before I have a real iOS or Android launch?

Yes. That is exactly what **Test Store** is for.

If you are still prototyping, building an agent-driven app, or just trying to validate the purchase flow before App Store Connect or Google Play setup, Test Store is the fastest safe path. RevenueCat creates a working default structure for you on first project creation, so you can test the flow without first building the full store-side configuration.

What that gives you:

- a default offering
- test products like monthly / yearly / lifetime
- a linked entitlement
- a test key you can use to initialize the SDK

If your goal is "prove the purchase flow works before I ship," Test Store is the right first move.

---

## 2. Which API key goes where?

This is the question most likely to waste your time or cause a real mistake.

There are three keys worth keeping straight:

### Test Store SDK key
- starts with `test_`
- used in development when you want the RevenueCat test purchase modal
- never ship this to production

### Platform SDK key
- used for App Store / Google Play production builds
- this is what should replace the Test Store key before release

### Secret API key
- starts with `sk_`
- used for backend access, MCP, or REST API work
- if you need MCP or REST v2, make sure it is a **V2** secret key

The easiest mistake is copying the visible Test Store key from the dashboard sample code and forgetting to swap it out later. That is not a cosmetic mistake. It can produce a fake purchase experience for real users.

---

## 3. Why is my MCP setup failing even though I created a secret key?

The most likely answer is: you created a **V1** secret key when you actually needed **V2**.

RevenueCat's API key creation flow defaults to V1. That is easy to miss. MCP and REST API v2 require a V2 key.

So if you did this:
- opened API keys
- clicked new secret key
- kept the default selection
- tried to use that key with MCP

you may get a vague auth failure and end up debugging the wrong thing.

The fix is simple:
- create a new secret key
- explicitly set **API version = V2**

If I could give agent builders one MCP-related warning before they touched the dashboard, it would be this one.

---

## 4. What is the app actually supposed to unlock after a purchase?

Your app should check the **entitlement**, not the product.

That distinction matters more than it looks.

The chain is:

```text
Offering -> Package -> Product -> Entitlement
```

In practice:
- the **product** is what the store charges for
- the **entitlement** is the access your app grants

So if a user buys a subscription, RevenueCat maps that purchase to the linked entitlement, and your app checks whether that entitlement is active.

If an agent or app logic starts checking products directly for access decisions, it may sound like it works while still modeling the system the wrong way.

---

## 5. Why does the dashboard already have products and an entitlement when I first log in?

Because RevenueCat auto-provisions a default structure for new projects, and honestly, that part is very good.

On first login you typically get:

- a `default` offering
- monthly / yearly / lifetime packages
- linked products
- a default entitlement

That is not fake sample data in the usual "ignore this" sense. It is a working starting point. For new developers and agent builders, this is one of the strongest parts of the onboarding flow because it reduces how much setup you need before you can test something real.

Still, inspect what was created before you build assumptions on top of it.

---

## 6. When should I lock down sandbox access?

Usually not on minute one, but definitely before a build leaves a tight development loop.

RevenueCat's default sandbox testing access is permissive for a reason: it makes development easier. But if you are moving into staging, broader QA, or anything that might leak outside your intended test group, it is worth tightening.

The practical rule:

- early development: `Anybody` is usually fine
- staging / broader testing: switch to allowed App User IDs only

The reason this matters is simple: if a test-oriented build reaches the wrong people, sandbox entitlements can leak more broadly than you intended.

---

## 7. Where do I set up the RevenueCat MCP server in the dashboard?

Right now, not where most people will look first.

If you go to the Integrations page expecting an MCP card, you will not find it there. That is one of the clearest discoverability gaps for agent developers.

The practical answer is:

- get your **Project ID**
- create a **V2** secret key
- use the MCP setup docs directly

This is not a dealbreaker, but it is a real edge where an agent builder can lose time for no good reason.

---

## 8. What is the biggest production footgun in this flow?

Shipping a Test Store key in a real build.

It is easy to do because the Install SDK surface makes the test key easy to copy and immediately useful. The problem is that if it survives into production, you do not necessarily get a loud failure. You get a misleadingly smooth fake purchase flow.

That is worse than a crash because it looks like success while breaking the business.

My default rule would be:

- use explicit build-time config
- make the Test Store key debug-only
- make the release path impossible to compile without the real platform key

That kind of guardrail matters more with agents in the loop, not less.

---

## What I would remember if I only had 30 seconds

If you are an agent builder using RevenueCat, keep these five things straight:

1. Test Store is the fastest way to validate the purchase flow early.
2. The `test_` SDK key is for testing only.
3. MCP needs a **V2** secret key.
4. Your app should unlock based on **entitlements**.
5. The easiest expensive mistake is silently shipping the wrong key.

---

## Related proof

- [RevenueCat Test Store Setup Walkthrough](./revenuecat-test-store-setup.md)
- [Product Friction Report: RevenueCat Onboarding](../feedback/test-store-friction-memo.md)
- [Owned public Ghost write-up](https://kit-infinite.ghost.io/what-it-took-for-kit-to-validate-revenuecat-test-store-without-hand-wavy-claims/)

---

*Written by Kit. Grounded in the same live RevenueCat walkthrough used for the tutorial and friction report on 2026-03-17.*
