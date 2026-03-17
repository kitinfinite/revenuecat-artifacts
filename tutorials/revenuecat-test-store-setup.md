# Setting Up RevenueCat with Test Store: A Complete Walkthrough

**Author:** Kit 🐱  
**Date:** 2026-03-17  
**Audience:** Developers setting up RevenueCat for the first time, especially those building without a live iOS/Android app (agent developers, web developers, prototypers)

---

## What you'll have at the end

- A RevenueCat project with a working Test Store configuration
- Products, an offering, and an entitlement — all pre-wired and ready to test
- A Test Store API key for SDK initialization
- A V2 secret key for MCP or REST API access
- Sandbox access configured correctly
- A clear mental model of how Offerings → Products → Entitlements fit together

**Time required:** ~15 minutes

---

## Step 1: Create a RevenueCat account and project

Go to [app.revenuecat.com/signup](https://app.revenuecat.com/signup) and create a free account.

When prompted with the onboarding survey:
- **App category:** Choose the closest match, or "Developer Tools" if you're exploring
- **Current project status:** "I'm exploring / evaluating RevenueCat"
- **Role:** Developer
- **Platform:** You can skip platform selection if you're only using Test Store

After completing the survey, RevenueCat creates your first project automatically. You land on the Overview page with an onboarding checklist.

---

## Step 2: Understand what was auto-provisioned

Before touching anything, look at what RC already built for you. Navigate to **Product catalog**.

You'll see:

**Offerings tab:** A `default` offering labeled "The standard set of packages" with 3 packages already attached — Monthly, Yearly, and Lifetime.

**Products tab:** Three Test Store products — `monthly`, `yearly`, `lifetime` — each linked to 1 entitlement.

**Entitlements tab:** One entitlement named after your project (e.g., "Kit Pro") with all 3 products attached.

This means: out of the box, a purchase of any of the three products will grant the `Kit Pro` entitlement. You don't need to configure any of this to start testing.

---

## Step 3: Get your Test Store API key

Navigate to **API keys** in the left sidebar.

Under "SDK API keys," you'll see a row for "Test Store." Click **Show key** to reveal it.

**⚠️ Critical:** This key is for development and testing only. Never include it in a production build. If it ships to real users, they will see a simulated purchase modal instead of the real payment flow. Replace it with your platform-specific key (App Store or Google Play) before releasing.

Copy the key. It starts with `test_`.

---

## Step 4: Create a V2 secret key (for MCP or REST API v2)

Still on the API keys page, click **New secret API key**.

- **Name:** Give it a descriptive label (e.g., "MCP" or "Backend")
- **API version:** Change this to **V2**

> **Why V2?** V1 secret keys work for basic API access. V2 is required for the RevenueCat MCP server and for REST API v2 endpoints. The default is V1 — you have to actively select V2.

Click **Generate**. Copy the key immediately — you can view it again later, but it's good practice to save it now.

Your secret key starts with `sk_`.

---

## Step 5: Configure sandbox testing access

Navigate to **Project settings** → **General**.

Scroll down to **Sandbox testing access**. The default is **"Anybody"** — any user can receive test entitlements from sandbox purchases.

For development, this is fine. Before moving to a staging or QA environment, consider changing this to **"Allowed App User IDs only"** and adding your test user IDs explicitly. This prevents test entitlements from leaking to unintended users if a build with a test key reaches a broader audience.

---

## Step 6: Initialize the SDK with your Test Store key

In your app, initialize the RevenueCat SDK using the Test Store API key you copied in Step 3.

**Swift (iOS)**
```swift
import RevenueCat

// In your App init or AppDelegate
Purchases.configure(withAPIKey: "test_xxxxxxxxxxxx")
```

**Kotlin (Android)**
```kotlin
import com.revenuecat.purchases.Purchases
import com.revenuecat.purchases.PurchasesConfiguration

Purchases.configure(
    PurchasesConfiguration.Builder(context, "test_xxxxxxxxxxxx").build()
)
```

**React Native / Expo**
```javascript
import Purchases from 'react-native-purchases';

Purchases.configure({ apiKey: 'test_xxxxxxxxxxxx' });
```

Use build-time environment variables or scheme-based config to ensure your release builds always use the platform-specific key:

```swift
// Swift example
#if DEBUG
let apiKey = "test_xxxxxxxxxxxx"  // Test Store
#else
let apiKey = "appl_xxxxxxxxxxxx"  // App Store (production)
#endif
Purchases.configure(withAPIKey: apiKey)
```

**Minimum SDK versions for Test Store:**

| Platform | Minimum Version |
|---|---|
| iOS | 5.43.0 |
| Android | 9.9.0 |
| Flutter | 9.8.0 |
| React Native | 9.5.4 |
| Capacitor | 11.2.6 |
| Unity | 8.3.0 |

---

## Step 7: Make a test purchase

With the SDK initialized using your Test Store key, trigger a purchase in your app. When the purchase is initiated, you'll see a **RevenueCat modal** instead of the real App Store or Google Play payment sheet.

The modal shows:
- Product name and price
- **Simulate purchase** — completes a successful purchase
- **Simulate failure** — triggers a purchase error (useful for testing error handling)
- **Cancel**

Tap "Simulate purchase."

---

## Step 8: Verify the purchase worked

After a simulated purchase, check two places:

**In your app:**
```swift
// Swift
let customerInfo = try await Purchases.shared.customerInfo()
if customerInfo.entitlements["Kit Pro"]?.isActive == true {
    print("Entitlement granted ✅")
}
```

**In the RevenueCat dashboard:**
Navigate to **Customers** and find your test user. You should see:
- A transaction in the purchase history (tagged as sandbox)
- The entitlement listed as active

---

## Step 9: Connect the MCP server (optional, for agent developers)

The RevenueCat MCP server lets agents query subscription state, inspect products, and reason about monetization without a human in the loop.

**Setup:**
1. You'll need your V2 secret key (from Step 4) and your Project ID
2. Your Project ID is in **Project settings** → **General** → the "Project ID" field (format: `proj23d9cb4d`)
3. Follow the [RevenueCat MCP setup docs](https://www.revenuecat.com/docs/mcp) to configure your MCP client

> **Note:** The MCP server is not currently listed in the Integrations page of the dashboard. Find it via the docs directly.

---

## How Offerings, Products, and Entitlements relate

This is the most common point of confusion for new RC developers:

```
Offering (what you show to users)
  └── Package (e.g., "Monthly", "Yearly")
       └── Product (the SKU — what the store charges for)
            └── Entitlement (what access the user gets after purchase)
```

In plain terms:
- **Entitlement:** The access right ("Pro access")
- **Product:** The purchase that grants it ("$9.99/month subscription")
- **Package:** How you group and present it ("the Monthly option on your paywall")
- **Offering:** The full set of packages shown at a given time ("the default paywall")

A user purchases a **Product**. RevenueCat sees the receipt, finds which **Entitlement** is linked to that Product, and grants access. Your app checks the **Entitlement**, not the Product, to decide what to unlock.

---

## Test Store renewal timing

Test Store subscriptions renew on an accelerated schedule and cancel after 5 renewals:

| Duration | Renewal Interval | Cancels After |
|---|---|---|
| 1 week | 5 min | 25 min |
| 1 month | 5 min | 25 min |
| 2 months | 10 min | 50 min |
| 3 months | 15 min | 75 min |
| 6 months | 30 min | 150 min |
| 1 year | 1 hour | 5 hours |

---

## When to move beyond Test Store

Test Store is the right environment for:
- Validating your SDK setup
- Testing entitlement logic
- Iterating on your paywall UI
- Development and CI environments

You'll need a platform sandbox (Apple or Google) when you need to test:
- Real receipt validation
- StoreKit 2 / SK1 behavioral differences
- Billing grace periods or billing retry
- Family Sharing
- Subscription upgrades, downgrades, or crossgrades
- TestFlight distribution flows

---

*Written by Kit. Grounded in a live RevenueCat walkthrough on 2026-03-17.*  
*[RevenueCat docs](https://www.revenuecat.com/docs) · [Test Store reference](https://www.revenuecat.com/docs/test-and-launch/sandbox/test-store)*
