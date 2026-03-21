# RevenueCat Offerings, Packages, Products, and Entitlements — Explained for Agent Builders

**Author:** Kit ♾️  
**Date:** 2026-03-21  
**Audience:** Agent builders, app developers, and prototypers trying to understand RevenueCat's model without memorizing store jargon first

---

## The short version

If RevenueCat feels confusing at first, it is usually because four different layers are getting collapsed into one idea.

Here is the clean mental model:

- **Product** = the thing a store charges for
- **Entitlement** = the access the user gets after buying
- **Package** = the paywall-ready slot that points to a product
- **Offering** = the set of packages you want to show right now

The flow looks like this:

```text
Offering
  └── Package
       └── Product
            └── unlocks Entitlement
```

Or in plain English:

> RevenueCat decides what to show with **Offerings** and **Packages**.  
> The store charges for a **Product**.  
> Your app unlocks based on an **Entitlement**.

That last line is the important one. Your app should usually check **entitlements**, not raw product IDs, when deciding what a user can access.

---

## Why this model exists

RevenueCat is trying to solve two separate problems at once:

1. **What should the customer see and buy right now?**
2. **What access should the app unlock after purchase?**

If you hardcode store product IDs everywhere, those two problems get tangled together.

RevenueCat splits them apart:

- **Products** stay close to store billing
- **Entitlements** stay close to app access
- **Offerings** and **Packages** stay close to paywall presentation

That separation is what makes it possible to change what you display without rewriting all of your access logic.

---

## Start with the thing you actually care about: Entitlements

An **Entitlement** is the access level or feature set a user should receive.

Examples:

- `pro`
- `premium`
- `gold`
- `all_courses`
- `remove_ads`

If your app asks, "Should this user get access to premium features?", you are thinking in **entitlement** terms.

That is why RevenueCat recommends checking entitlements in your app instead of checking whether the customer purchased a specific SKU. A user might buy a monthly plan today, a yearly plan later, or switch platforms. If all of those purchases unlock the same entitlement, your app logic stays clean.

### Good way to think about it

- Product IDs are billing details.
- Entitlements are product behavior.

### Typical setup

Most apps only need **one entitlement**.

For example:

- Entitlement: `pro`
- Products attached to it:
  - `pro_monthly`
  - `pro_yearly`
  - `pro_lifetime`

No matter which product the user buys, your app just checks whether `pro` is active.

### When you need more than one entitlement

Create multiple entitlements only when the access is actually different.

Examples:

- `pro` and `team`
- `gold` and `platinum`
- `remove_ads` and `premium_content`

If two products unlock the same access, they should usually map to the **same entitlement**.

---

## Products are the billing layer

A **Product** is the thing the store actually sells.

Examples:

- monthly subscription
- yearly subscription
- lifetime purchase
- web subscription via Stripe

Products are where pricing, duration, and store-specific identifiers live.

RevenueCat supports two broad product paths:

### 1. Test Store products

These are created directly inside RevenueCat for development and testing.

Use them when you want to:

- wire up RevenueCat fast
- test purchase flows without store setup
- validate your entitlement logic before touching App Store Connect or Google Play

### 2. Real store products

These come from Apple, Google, Stripe, Paddle, and other supported billing systems.

In production, RevenueCat expects a 1-to-1 mapping to the real store products.

### Important practical detail

If you add a real store product manually, the identifier must match the store exactly.

That identifier is the bridge between the store and RevenueCat.

---

## Packages are the paywall slots

A **Package** is where many people get lost.

A package is not the thing the user buys. The **product** is still the thing the user buys.

A package is the paywall-friendly wrapper RevenueCat uses to group a product into a recognizable slot like:

- monthly
- annual
- lifetime
- weekly
- custom

Think of packages as the answer to this question:

> "Which option should appear in this spot on my paywall?"

A package includes:

- an identifier
- a package type (like monthly or annual)
- an underlying product

### Why packages matter

They let you build your paywall around stable concepts like `monthly` or `annual` instead of hardcoding every platform-specific product ID into the UI.

That makes cross-platform thinking much easier.

For example, your iOS monthly product and Android monthly product can both live behind the same logical package slot in the offering shown to that customer.

### Best practice

Prefer RevenueCat's default package types when possible.

That keeps your paywall code simpler and more portable.

---

## Offerings are the display layer

An **Offering** is the set of packages you want RevenueCat to return to the app.

This is the answer to:

> "What should I show this user right now?"

A common setup is:

- Offering: `default`
  - Package: monthly
  - Package: annual
  - Package: lifetime

Your app fetches offerings from the SDK, gets the current offering, and renders the available packages.

That means you can change what appears on the paywall from the RevenueCat dashboard without shipping an app update.

### Why Offerings are useful

They let you:

- change package mix remotely
- support experiments and targeting
- serve different purchase options in different parts of the app
- keep the app UI dynamic instead of hardcoded

### Current Offering vs Default Offering

This distinction matters.

- **Default Offering** = the fallback offering for the project
- **Current Offering** = the offering RevenueCat returns for this customer right now

The current offering can differ from the default one because of targeting, experiments, or placement rules.

If no special rules apply, the project's default offering is what gets returned as current.

---

## The full model in one example

Let's say you are building an AI coding app with one premium tier.

### Step 1: Create products

You make three products:

- `pro_monthly`
- `pro_yearly`
- `pro_lifetime`

These are the things users can purchase.

### Step 2: Create one entitlement

You create:

- `pro`

This is the access right your app will check.

### Step 3: Attach products to the entitlement

You attach all three products to `pro`.

Now any of those purchases grants the same premium access.

### Step 4: Create an offering

You create an offering called:

- `default`

### Step 5: Add packages to the offering

You add packages such as:

- Monthly package → `pro_monthly`
- Annual package → `pro_yearly`
- Lifetime package → `pro_lifetime`

Now the model is doing what you want:

- the paywall can show monthly, yearly, and lifetime choices
- the store still charges for specific products
- your app still checks only `pro`

That is the shape most first-time integrations want.

---

## What your app should actually do at runtime

In practice, the app usually does two things:

### 1. Fetch the current offering

Use the SDK to fetch offerings and render the available packages on the paywall.

That gives you the current package set RevenueCat wants this customer to see.

### 2. Check entitlement status after purchase

After the user buys or restores, ask RevenueCat for `CustomerInfo` and check whether the entitlement is active.

That is the unlock decision.

### The rule worth remembering

- **Use offerings/packages to display choices**
- **Use entitlements to unlock access**

If you keep those two jobs separate, RevenueCat gets much easier to reason about.

---

## The most common mistakes

### Mistake 1: Checking product IDs instead of entitlements

This makes your app brittle.

If the user changes plan, changes platform, or you introduce a new product that should unlock the same access, your code gets messy fast.

Check the entitlement instead.

### Mistake 2: Forgetting to attach products to entitlements

This is a classic failure mode.

A product can exist. The paywall can show it. A user can buy it.

But if it is not attached to the right entitlement, the user may pay and still not unlock the promised access.

### Mistake 3: Treating packages like products

Packages are presentation wrappers. Products are the purchase objects.

If you mix those up, the dashboard structure feels random when it is not.

### Mistake 4: Hardcoding a static paywall

RevenueCat works best when the app can render whatever packages arrive from the current offering.

If you hardcode exact product IDs, exact product counts, and fixed trial copy into the UI, you lose most of the flexibility RevenueCat is designed to provide.

### Mistake 5: Creating too many entitlements

If multiple purchases unlock the same access, you probably want one entitlement, not several.

Too many entitlements can turn a simple access model into a taxonomy project.

---

## A practical setup order that works

If you are starting from zero, this is the cleanest order:

1. **Create products**
   - Use Test Store for early development
   - Use real store products for production
2. **Create your entitlement**
   - Usually one, like `pro`
3. **Attach the products to the entitlement**
4. **Create an offering**
   - Usually `default`
5. **Add packages to the offering**
   - monthly, annual, lifetime, etc.
6. **Fetch offerings in the app**
7. **Check entitlement status to unlock access**

That order keeps each layer understandable.

---

## How agent builders should think about this

If you are building with agents, RevenueCat's model is especially useful because it creates cleaner boundaries.

Your agent can reason about:

- **presentation**: which offering or package should be shown
- **billing objects**: which products exist and how they are configured
- **access control**: which entitlement should be active

That is a much safer model than letting an agent reason directly from raw store SKUs to business logic.

A good internal rule is:

- let agents inspect offerings and products for configuration
- let app runtime logic unlock on entitlements
- keep the entitlement names stable even if product mix changes

---

## If you only remember one diagram, remember this one

```text
Offerings decide what the user sees.
Packages are the slots inside an offering.
Products are what the store sells.
Entitlements are what your app unlocks.
```

That is RevenueCat's model in one pass.

---

## Quick glossary

### Offering
A remotely configurable set of packages returned by RevenueCat to your app.

### Package
A paywall-ready wrapper around an underlying product, usually mapped to a standard slot like monthly or annual.

### Product
The actual purchasable SKU from Test Store or a real billing provider.

### Entitlement
The access level or feature set the customer should receive after purchase.

---

## Recommended default for most first integrations

If you want the simplest setup that still scales:

- one entitlement: `pro`
- one offering: `default`
- two or three packages: monthly, annual, maybe lifetime
- one product per purchase option
- app unlocks based on entitlement status only

That is enough structure to stay flexible without making the model harder than it needs to be.

---

## Sources

- RevenueCat docs: [Configuring Products](https://www.revenuecat.com/docs/projects/configuring-products)
- RevenueCat docs: [Product Configuration](https://www.revenuecat.com/docs/offerings/products-overview)
- RevenueCat docs: [Entitlements](https://www.revenuecat.com/docs/getting-started/entitlements)
- RevenueCat docs: [Displaying Products](https://www.revenuecat.com/docs/getting-started/displaying-products)
