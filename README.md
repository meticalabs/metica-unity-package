<div align="center">

  <!-- Light mode -->
  <img src="docs/Metica_Logomark_Colour_Light_Mode.png#gh-light-mode-only" 
       alt="Metica Logo" width="200"/>

  <!-- Dark mode -->
  <img src="docs/Metica_Logomark_Colour_Dark_Mode.png#gh-dark-mode-only" 
       alt="Metica Logo" width="200"/>

  <br><br>
</div>

# Metica Unity SDK

The Metica Unity Package provides the complete Unity SDK used to integrate Metica services into your Unity game. 
This repository contains the official, versioned Unity package distributed via GitHub.

[Here](https://github.com/meticalabs/metica-unity-package/releases) you can find the latest releases.

# Metica Analytics for Unity

Metica Analytics is a feature of Metica SDK which allows you to track user events, purchases, sessions, and custom data in your Unity application.

## Prerequisites

Before using Metica Analytics, you need:

1. **API Key** – obtained from the [Metica platform](https://metica.com)
2. **App ID** – obtained from the Metica platform
3. A **Max SDK Key**, obtainable from Applovin platfom (only required when initializing the SDK with ads)

## Installation

### 1. Install Metica SDK

1. Download the `.unitypackage` from the [Metica GitHub repository](https://github.com/meticalabs/metica-unity-package)
2. In Unity Editor, navigate to **Assets → Import Package → Custom Package**
3. Select the downloaded `.unitypackage` file

### 2. Install Analytics Package

The Analytics feature requires an additional package to be installed:

1. Download [`com.metica.analytics.abstractions-1.0.0.tgz`](https://drive.google.com/file/d/1H6WYJqsIjQBls3H3giZoEKnIdDje4-Lj/view?usp=sharing)
2. Open Unity Editor
3. Go to **Window → Package Manager**
4. Click the **+** button in the top-left corner
5. Select **Add package from tarball...**
6. Navigate to and select `com.metica.analytics.abstractions-1.0.0.tgz`
7. Click **Open**

## Initialization

Analytics requires the Metica SDK to be initialized first. The SDK supports two initialization flows that share the same `MeticaInitConfig`.

### Standard initialization

A single call to `MeticaSdk.InitializeAsync` (or `MeticaSdk.Initialize`) with mediation info initializes both Ads and Analytics together. If your game does not need to track analytics events before the ads SDK is ready, this is all you need:

```csharp
using Metica;

var config = new MeticaInitConfig("YOUR_API_KEY", "YOUR_APP_ID", "YOUR_USER_ID");
var mediationInfo = new MeticaMediationInfo(MeticaMediationType.MAX, "YOUR_MAX_SDK_KEY");

// Initializes both Ads and Analytics
var initResponse = await MeticaSdk.InitializeAsync(config, mediationInfo);
```

### Two-stage initialization (early analytics)

Some integrations need Analytics to be available earlier in the app lifecycle than the rest of the SDK — the standard initialization call is often deferred until later in the boot flow (waiting on user consent, on a loading or splash screen, on remote configuration, or simply to defer ads bring-up until the game is ready). While that call is pending, early-session events such as `install` and `sessionStart` would otherwise be lost, because the SDK's internal event queue only activates after init.

To avoid this, you can pre-initialize Analytics with `MeticaSdk.InitializeAnalytics(config)` as soon as the app starts. The internal event queue activates immediately, so early events are buffered and delivered once the SDK is ready. Later, when ads are needed, call `MeticaSdk.InitializeAsync` (or `MeticaSdk.Initialize`) with mediation info to add Ads on top of the already-initialized Analytics:

```csharp
using Metica;

// Step 1 — at app start: Analytics-only, synchronous
var config = new MeticaInitConfig("YOUR_API_KEY", "YOUR_APP_ID", "YOUR_USER_ID");
MeticaSdk.InitializeAnalytics(config);

// ... fire early-session analytics events here (install, sessionStart, ...) ...

// Step 2 — after consent / when ads are needed: add Ads on top of Analytics
var mediationInfo = new MeticaMediationInfo(MeticaMediationType.MAX, "YOUR_MAX_SDK_KEY");
var initResponse = await MeticaSdk.InitializeAsync(config, mediationInfo);
```

`MeticaSdk.InitializeAnalytics` is **optional** — if your integration does not need early analytics, the standard one-call flow continues to work exactly as before.

> **Caveat:** when both calls are used, they should pass the same `MeticaInitConfig` (`ApiKey`, `AppId`, `UserId`). If the second call supplies a different config, the native SDK keeps the configuration from the first call and logs a warning — the differing values are ignored.

> **Note:** The `YOUR_USER_ID` parameter is optional. If omitted, the SDK generates a unique UUID automatically.

For the full SDK documentation, see: [Metica Unity SDK Documentation](https://docs.metica.com/api/unity-sdk/unity-sdk-2)

## Using Analytics

After initialization, access the Analytics API via `MeticaSdk.Analytics`:

```csharp
using Metica.Analytics.Abstractions;

var analytics = MeticaSdk.Analytics;
```

## Environment Attributes

In addition to the properties you supply per event, the SDK automatically attaches a set of **environment attributes** describing the device, app, and session. These are collected by the native layer and included with every analytics event — you do not need to send them explicitly.

The table below lists each attribute, its type, and the platforms on which it is populated:

| Attribute            | Type     | Android | iOS | Description                                                              |
|----------------------|----------|:-------:|:---:|--------------------------------------------------------------------------|
| `appVersion`         | string   | ✅      | ✅  | Host app's version (e.g. `1.4.2`).                                       |
| `meticaUnitySdk`     | string   | ✅      | ✅  | Version of the Metica SDK in use.                                        |
| `platform`           | string   | ✅      | ✅  | `android` or `ios`.                                                      |
| `osVersion`          | string   | ✅      | ✅  | Operating system version (e.g. `14`, `17.4`).                            |
| `buildNumber`        | string   | ✅      | ✅  | OS build identifier (Android `Build.ID`, iOS build number).              |
| `deviceModel`        | string   | ✅      | ✅  | Device model identifier (e.g. `Pixel 7`, `iPhone15,2`).                  |
| `deviceManufacturer` | string?  | ✅      | —   | Hardware manufacturer (e.g. `Google`, `Samsung`). Android only.          |
| `deviceType`         | string   | ✅      | ✅  | `phone` or `tablet`.                                                     |
| `deviceMemory`       | long     | ✅      | ✅  | Total device RAM in bytes.                                               |
| `locale`             | string   | ✅      | ✅  | BCP-47 language tag (e.g. `en-US`).                                      |
| `localTimezone`      | string   | ✅      | ✅  | IANA time zone identifier (e.g. `Europe/London`).                        |
| `country`            | string?  | ✅      | ✅  | ISO 3166-1 alpha-2 country code, lowercased.                             |
| `storeCountry`       | string?  | ✅      | ✅  | App store storefront country (when available).                           |
| `sessionCount`       | long     | ✅      | ✅  | Number of sessions started by this user on this device.                  |
| `lastSessionLength`  | long     | ✅      | ✅  | Duration of the previous session, in milliseconds.                       |
| `gaid`               | string?  | ✅      | —   | Google Advertising ID (lowercased). Android only; null when unavailable. |
| `idfa`               | string?  | —       | ✅  | Apple Identifier For Advertisers. iOS only; null when unauthorized.      |
| `idfv`               | string?  | —       | ✅  | Apple Identifier For Vendor. iOS only.                                   |

> Attributes marked nullable (`?`) may be absent in the payload when the platform cannot resolve them (e.g. `gaid`/`idfa` when the user has not granted tracking consent).

## Data Constraints

### Property Limits

- **Maximum properties:** `customPayload` and user attributes can contain up to **100 properties**
- **Array size limit:** Arrays can contain up to **100 elements** of the same primitive type
- **Nested object:** Nested objects are no permitted

### Allowed Data Types

Only primitive types are accepted in `customPayload` and user attributes:

| Type | Example |
|------|---------|
| `string` | `"hello"` |
| `number` | `42`, `3.14` |
| `boolean` | `true`, `false` |
| `array` | `[1, 2, 3]`, `["a", "b"]` |

### Property Type Consistency

**Once a property is sent with a specific type, that type cannot be changed.**

If your game sends a property (in any event type), all future events from your game—or any other game in your organization—must use the same type for that property.

```csharp
// First event sets "score" as a number
analytics.LogCustomEvent("levelComplete", new Dictionary<string, object>
{
    ["score"] = 100  // number type
});

// All future events MUST send "score" as a number
// Sending ["score"] = "100" (string) will cause ingestion errors
```

If you need to change a property's type:
1. Stop sending the original property
2. Create a new property with a different name

Violating type consistency will cause **errors and missing data** in the ingestion pipeline.

## Event Types

### Custom Event

Track any custom event with arbitrary properties:

```csharp
analytics.LogCustomEvent(
    eventName: "puzzleLevelEndOfferImpression",
    properties: new Dictionary<string, object>
    {
        ["genre"] = "puzzle",
        ["levelIndex"] = 42,
        ["stars"] = 3,
        ["remainingMoves"] = 5,
        ["cohort"] = "experiment_a",
    }
);
```

**Custom event naming rules:**

- Event names **must be camelCase** (e.g., `levelComplete`, `itemPurchased`)
- The following names are **reserved** and cannot be used:
  - `purchase`
  - `impression`
  - `sessionStart`
  - `install`
  - `fullStateUpdate`
  - `partialStateUpdate`

### Purchase Event

Track in-app purchases:

```csharp
analytics.LogPurchaseEvent(
    productId: "com.game.gems_100",
    currency: "USD",
    amount: 4.99,
    status: "completed",
    errorCode: null,
    referenceId: "txn_abc123",
    customPayload: new Dictionary<string, object>
    {
        ["source"] = "shop",
        ["discountApplied"] = true,
    }
);
```

### Session Start Event

Track when a user session begins:

```csharp
analytics.LogSessionStartEvent(
    customPayload: new Dictionary<string, object>
    {
        ["entryPoint"] = "notification",
    }
);
```

### Install Event

Track app installations:

```csharp
analytics.LogInstallEvent(
    customPayload: new Dictionary<string, object>
    {
        ["campaignId"] = "summer_promo_2024",
    }
);
```

### Impression Event

Track ad impressions:

```csharp
analytics.LogImpressionEvent(
    value: 0.00555,
    type: "Rewarded",
    mediator: "AppLovin",
    source: "ironSource",
    placement: "LevelSuccess",
    customPayload: new Dictionary<string, object>
    {
        ["levelIndex"] = 10,
    }
);
```

### Full State Update Event

Send a complete snapshot of user attributes. This **replaces** all existing attributes previously sent:

```csharp
analytics.LogFullStateUpdateEvent(
    attributes: new Dictionary<string, object>
    {
        ["playerLevel"] = 25,
        ["totalCoins"] = 15000,
        ["isPremium"] = true
    }
);
```

### Partial State Update Event

Update specific user attributes without replacing the full state:

```csharp
analytics.LogPartialStateUpdateEvent(
    attributes: new Dictionary<string, object>
    {
        ["totalCoins"] = 16500,
    }
);
```

## Building a Custom Analytics Wrapper

Studios that ship many titles — often grouped by genre (puzzle, simulation, arcade, …) — frequently want a **shared, internal analytics library** that standardises how each game describes its world while still routing events through Metica. The Metica SDK is designed to support this pattern through the **`Metica.Analytics.Abstractions`** package.

### Architecture

The integration is split across two layers:

| Layer                              | Owner       | Changes Often? | Purpose                                                                                  |
|------------------------------------|-------------|----------------|------------------------------------------------------------------------------------------|
| `Metica.Analytics.Abstractions`    | Metica      | Rarely         | Tiny, stable contract — `IMeticaAnalytics` interface + minimal DTOs.                     |
| Metica Unity SDK                   | Metica      | Often          | Full implementation — transport, retries, device info, ad mediation, etc.                |
| Your custom wrapper library        | Your studio | As needed      | Genre/title-specific helpers that map gameplay state to event payloads.                  |

Your custom library depends only on `Metica.Analytics.Abstractions`, **not** on the full Unity SDK. As long as the interface stays backward-compatible (which it is designed to), SDK upgrades do not require you to rebuild or change your wrapper code.

### The stable contract

Your wrapper code targets the `IMeticaAnalytics` interface exposed by the abstractions package:

```csharp
namespace Metica.Analytics.Abstractions
{
    public interface IMeticaAnalytics
    {
        void LogPurchaseEvent(
            string productId, string currency, double amount, string status,
            string? errorCode, string? referenceId,
            Dictionary<string, object>? customPayload);

        void LogSessionStartEvent(Dictionary<string, object>? customPayload);
        void LogInstallEvent(Dictionary<string, object>? customPayload);

        void LogImpressionEvent(
            double value, string type, string mediator, string source,
            string? placement, Dictionary<string, object>? customPayload);

        void LogFullStateUpdateEvent(Dictionary<string, object> attributes);
        void LogPartialStateUpdateEvent(Dictionary<string, object> attributes);

        void LogCustomEvent(string eventName, Dictionary<string, object>? properties);
    }
}
```

`MeticaSdk.Analytics` (returned after initialization) implements this interface — your wrapper accepts an `IMeticaAnalytics` in its constructor and never references the concrete SDK.

### Example: a genre-specific wrapper

A puzzle-genre wrapper might encapsulate the player context once and produce strongly-typed event helpers:

```csharp
using System.Collections.Generic;
using Metica.Analytics.Abstractions;

public class PuzzlePlayerContext
{
    public int PlayerLevel { get; set; }
    public int CurrentLevelIndex { get; set; }
    public int StarsEarned { get; set; }
    public int RemainingMoves { get; set; }
    public string ExperimentCohort { get; set; }
}

public class PuzzleAnalytics
{
    private readonly IMeticaAnalytics _analytics;

    public PuzzleAnalytics(IMeticaAnalytics analytics)
    {
        _analytics = analytics;
    }

    public void LogLevelEndOfferImpression(string offerId, PuzzlePlayerContext ctx)
    {
        var payload = new Dictionary<string, object>
        {
            ["genre"]          = "puzzle",
            ["offerId"]        = offerId,
            ["levelIndex"]     = ctx.CurrentLevelIndex,
            ["stars"]          = ctx.StarsEarned,
            ["remainingMoves"] = ctx.RemainingMoves,
            ["cohort"]         = ctx.ExperimentCohort,
        };

        _analytics.LogCustomEvent("puzzleLevelEndOfferImpression", payload);
    }
}
```

Wiring it up in your game:

```csharp
var config = new MeticaInitConfig("YOUR_API_KEY", "YOUR_APP_ID", "YOUR_USER_ID");
await MeticaSdk.InitializeAsync(config, mediationInfo);

var puzzle = new PuzzleAnalytics(MeticaSdk.Analytics);
puzzle.LogLevelEndOfferImpression("offer_123", currentCtx);
```

Different genres (simulation, arcade, idle, …) follow the exact same pattern — each studio team owns its own wrapper, with its own fixed property schema, while sharing the same underlying transport.

### Why this pattern

- **Compatibility across SDK upgrades** — wrappers depend on the small, stable interface, so a Metica SDK release does not force a rebuild of your internal library.
- **Schema consistency** — every game in the same genre emits the same property names and types, which keeps the [Property Type Consistency](#property-type-consistency) rule easy to satisfy across titles.
- **Discoverability** — game developers see a strongly-typed API (`LogLevelEndOfferImpression(...)`) instead of having to remember loose dictionary keys.
- **Testability** — wrappers can be unit-tested against a mock `IMeticaAnalytics`, with no SDK internals or native bridges involved.
- **Clear ownership** — Metica owns the interface contract; your studio owns how gameplay state maps to event payloads.

