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

Analytics requires the Metica SDK to be initialized first:

```csharp
using Metica;

var config = new MeticaInitConfig("YOUR_API_KEY", "YOUR_APP_ID", "YOUR_USER_ID");

// Initialize SDK
var initResponse = await MeticaSdk.InitializeAsync(config);
```

> **Note:** The `YOUR_USER_ID` parameter is optional. If omitted, the SDK generates a unique UUID automatically.

For the full SDK documentation, see: [Metica Unity SDK Documentation](https://docs.metica.com/api/unity-sdk/unity-sdk-2)

## Using Analytics

After initialization, access the Analytics API via `MeticaSdk.Analytics`:

```csharp
using Metica.Analytics.Abstractions;

var analytics = MeticaSdk.Analytics;
```

## Data Constraints

### Property Limits

- **Maximum properties:** `customPayload` and user attributes can contain up to **100 properties**
- **Array size limit:** Arrays can contain up to **100 elements** of the same primitive type

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

