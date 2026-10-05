# Unity Subscription System

**A Unity client template for account registration, login, and subscription management, with hosted Stripe-related payment pages opened through GPM WebView.**

This earlier integration project connects a Malay account interface to an external PHP service. It demonstrates the client side of an account/subscription workflow: submitting forms, displaying server responses, caching account details, refreshing subscription status, and opening hosted plan-management pages.

**Scope:** this repository contains the Unity client and bundled plugins. The PHP backend, database schema, hosted payment pages, and payment-provider configuration are not included. The source baseline reviewed here was committed in August 2024. Runtime and payment behavior were not tested for this documentation update; see [validation status](docs/VALIDATION.md).

## Features represented in the source

- Registration with name, email, phone, and password inputs, plus client-side form checks.
- Login and email-confirmation UI, account-detail display, and a request to send a verification email.
- A home screen with subscription plan, start/end dates, and status supplied by the backend.
- GPM WebView pages for choosing a plan and opening plan-management URLs.
- Periodic account/subscription refresh and scene routing between login and home.
- Logout requests and local preference cleanup.

The client stores account and subscription fields in `PlayerPrefs`. Those cached values support display and routing; this repository does not establish secure entitlement enforcement or payment confirmation on the server.

## Architecture

```text
Unity forms and account screens
  ├─ UnityWebRequest → external PHP account/subscription service
  ├─ PlayerPrefs    → local account/subscription display state
  └─ GPM WebView    → hosted plan-selection and management pages

Payment processing and provider configuration: external to this repository
```

The hosted pages and subscription records use Stripe-related identifiers. There is no direct Stripe SDK integration in the reviewed application scripts. A WebView page opening or closing is not recorded here as proof of a successful payment.

| Component | Responsibility |
| --- | --- |
| [`Assets/Domains.cs`](Assets/Domains.cs) | Shared backend base URL and persistent domain object |
| [`Assets/RegisterUserController.cs`](Assets/RegisterUserController.cs) | Registration form and request to `insertUser.php` |
| [`Assets/LoginController.cs`](Assets/LoginController.cs) | Login request, response handling, cached account fields, and initial subscription lookup |
| [`Assets/Player.cs`](Assets/Player.cs) | Account/subscription models, refresh requests, and logout |
| [`Assets/PlayerManager.cs`](Assets/PlayerManager.cs) | Periodic refresh and login/home scene routing |
| [`Assets/LoadPaymentSite.cs`](Assets/LoadPaymentSite.cs) | Plan selection/management URLs and WebView callbacks |
| [`Assets/loadDetails.cs`](Assets/loadDetails.cs) / [`loadSubscriptionDetails.cs`](Assets/loadSubscriptionDetails.cs) | Account, email-verification, and subscription presentation |
| [`Assets/SceneController.cs`](Assets/SceneController.cs) | Logout action and local preference reset |

## Technology and scenes

| Area | Tracked version or implementation |
| --- | --- |
| Editor | Unity **2022.3.14f1** |
| Application | C#, coroutines, UnityWebRequest, JsonUtility, PlayerPrefs |
| Interface | uGUI and TextMesh Pro **3.0.6** |
| Embedded pages | Bundled GPM WebView **2.0.2**, with Android and iOS native plugins |
| Verification tooling | Unity Test Framework **1.1.33** is declared; no application test suite was found |

Build Settings enable `Assets/Scenes/LogMasuk.unity` first, then `Assets/Scenes/LamanUtama.unity`. Native WebView behavior needs platform-specific testing; the presence of historical Windows build files does not establish that payment pages work on Windows.

## Review or adapt the project

1. Clone the source:

   ```sh
   git clone https://github.com/Azrinfreeman/SubscriptionSystemUnity.git
   ```

2. Add the repository root to Unity Hub and open it with **2022.3.14f1**. Let Unity restore packages and import assets.
3. Inspect `LogMasuk.unity` and the controllers before entering Play mode. `Domains.Awake` assigns an existing hosted backend URL in code; changing its Inspector field alone will not replace that assignment.
4. For an interactive demonstration, first supply a controlled test backend and update the domain in your local copy. There is no offline backend or mock-account mode bundled with this project. Cached login state can trigger recurring requests as soon as the app starts.
5. Use fictional accounts and provider test-mode pages for integration checks. Verify registration, login, email confirmation, subscription refresh, and WebView callbacks separately. Do not treat cached status as payment authorization.

The client references `loginVerify.php`, `insertUser.php`, `getinfouser.php`, `getinfo.php`, `logout.php`, `sendVerification.php`, and `pilihplan.php`. Their implementations and server response guarantees must be supplied separately; checkout, provider events, and entitlement behavior cannot be verified from this client alone.

Historical Windows player files are tracked under [`Publish/`](Publish/). They were not executed or compared with the current source for this review. No GitHub release was listed at review time.

## Integration limits to review

- Account identifiers, email/phone fields, subscription identifiers, and status are cached in PlayerPrefs. Review that data lifecycle when adapting the template.
- Hosted-page URLs include email or customer/price identifiers in query parameters; review how the external pages authenticate and validate those requests.
- Some request handlers log response text. Use fictional data when collecting demo logs.
- `PlayerManager.AssignInformation` currently reads both date fields from the `stripe_id` preference. That source-level issue is recorded in the validation notes; the separate subscription UI reads the date preferences directly.
- Logout invokes `PlayerPrefs.DeleteAll`, which matters if this template is embedded in a larger Unity application.

These are observations from the existing implementation. This documentation update changes no account, payment, or application behavior.

## Attribution

GPM is bundled with its [license notice](Assets/GPM/LICENSE.txt); the payment-page controller identifies its use of the GPM WebView example. Preserve third-party notices when adapting the project. This repository has no top-level software license, so confirm code and asset permissions before redistribution.
