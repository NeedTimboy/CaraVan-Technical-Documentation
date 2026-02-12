# CaraVan - Technical Documentation

**Version:** 0.4.1
**Platform:** Android (Native Kotlin)
**Min SDK:** 26 (Android 8.0 Oreo)
**Target SDK:** 35

---

## Overview

CaraVan is a native Android social app built for the van-life community. It combines a social feed, event coordination with maps, swipe-based dating, van build guides, and premium subscriptions into a single platform. The app uses on-device sample data for demonstration with Firebase integration planned for production.

---

## Tech Stack

### Core

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Kotlin | 2.1.0 |
| Build System | Gradle (Kotlin DSL) | AGP 8.7.3 |
| UI Framework | Material Design 3 | 1.12.0 |
| View Binding | Android ViewBinding | Built-in |
| Navigation | Jetpack Navigation Component | 2.7.6 |
| Async | Kotlin Coroutines | 1.7.3 |

### Firebase Services (BOM 32.7.0)

| Service | Purpose |
|---------|---------|
| Firebase Authentication | User sign-in and account management |
| Cloud Firestore | Document-based data storage |
| Cloud Storage | Media file hosting |
| Firebase Analytics | Usage metrics and event tracking |

### Google Play Services

| Service | Version | Purpose |
|---------|---------|---------|
| Google Maps SDK | 18.2.0 | Activity/event map with custom markers |
| Fused Location Provider | 21.1.0 | Device geolocation |

### Third-Party Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| RevenueCat Purchases | 9.19.4 | Subscription management and paywall |
| RevenueCat Purchases UI | 9.19.4 | Pre-built paywall screens |
| Glide | 4.16.0 | Image loading and caching |
| Lottie | 6.3.0 | Vector animations |

### Jetpack / AndroidX

| Library | Purpose |
|---------|---------|
| ConstraintLayout 2.2.0 | Responsive layouts |
| RecyclerView 1.3.2 | List/grid rendering |
| ViewPager2 1.0.0 | Swipeable page layouts |
| CardView 1.0.0 | Material card surfaces |
| Lifecycle (ViewModel, LiveData) 2.7.0 | Lifecycle-aware components |
| SplashScreen 1.0.1 | Android 12+ splash screen API |

---

## Architecture

### App Structure

CaraVan follows a single-Activity architecture with a fragment-based navigation pattern. `MainActivity` hosts a `BottomNavigationView` with five tabs, each loading a dedicated fragment.

```
Application
├── CareAVanApplication          # App-level initialization (RevenueCat, etc.)
│
├── Activities
│   ├── WelcomeActivity          # Animated splash / entry point
│   ├── OnboardingActivity       # Multi-step onboarding (ViewPager2)
│   ├── MainActivity             # Core shell (BottomNav + FragmentContainer)
│   ├── PostDetailActivity       # Full post view with comments
│   ├── UserProfileActivity      # Other user's profile
│   ├── FollowListActivity       # Followers / following list
│   ├── ChatActivity             # 1:1 messaging
│   ├── VideoPlayerActivity      # Video playback (MediaPlayer + SurfaceView)
│   ├── GuideDetailActivity      # Build guide detail with Q&A
│   ├── DatingProfileDetailActivity
│   └── PrivacyPolicyActivity
│
├── Fragments (Main Tabs)
│   ├── FeedFragment             # Social feed + messages
│   ├── CaravanFragment          # Map + calendar activities
│   ├── DatingFragment           # Swipe-based dating cards
│   ├── BuildHelpFragment        # Van build guides
│   └── AccountFragment          # User profile + settings
│
├── Fragments (Onboarding)
│   ├── OnboardingInviteFragment
│   ├── OnboardingPhotoNameFragment
│   ├── OnboardingUsernameFragment
│   └── OnboardingVanBioLocationFragment
│
├── Models
│   ├── UserProfile              # Community member profiles
│   ├── DatingProfile            # Dating card data
│   ├── Post                     # Feed posts with privacy levels
│   ├── Activity                 # Events/activities with location
│   ├── BuildGuide               # Educational build content
│   ├── GuideQuestion            # Q&A for guides
│   ├── Comment                  # Post comments
│   └── Message                  # Chat messages
│
├── Utilities
│   ├── DarkModeManager          # Centralized theme/color management
│   ├── UserSession              # In-memory session state (singleton)
│   ├── LocationSuggestions      # US location database (60+ cities)
│   ├── MentionHelper            # @mention text parsing and styling
│   ├── ReportHelper             # Content reporting system
│   └── CheckEntitlement         # RevenueCat entitlement verification
│
└── Views
    └── AnimatedGradientView     # Custom animated background
```

### Navigation Flow

```
WelcomeActivity
    │
    ├── [First Launch] → OnboardingActivity → MainActivity
    │
    └── [Returning User] → MainActivity
                              │
                              ├── Feed Tab
                              │   ├── My Feed (posts)
                              │   └── Messages (conversations)
                              │
                              ├── Activities Tab
                              │   ├── Map View (Google Maps)
                              │   └── Calendar View
                              │
                              ├── Dating Tab
                              │   ├── New Faces (swipe cards)
                              │   ├── Conversations
                              │   └── Preferences
                              │
                              ├── Build Help Tab
                              │   ├── Guide List (free + premium)
                              │   └── Ask a Builder
                              │
                              └── Account Tab
                                  ├── Profile Management
                                  ├── Settings
                                  └── Upgrade to FastTrack
```

### State Management

The app uses a singleton-based session model for in-memory state:

- **`UserSession`** (object singleton) - Holds the current user's profile data, preferences, dark mode flag, following/blocked lists, dating preferences, and created activities.
- **`MessageState`** (object singleton) - Tracks unread message count for badge display.
- **`DarkModeManager`** (object) - Provides centralized color resolution for light/dark themes. All fragments call into this to get the correct colors for text, cards, backgrounds, and accents.

Data is currently loaded from hardcoded sample sets in model companion objects. Firebase Firestore integration is configured for production use.

### Dark Mode

Dark mode is implemented through a custom `DarkModeManager` rather than Android's built-in DayNight theme. This gives fine-grained control over every UI element:

- Toggle is in the Settings dialog (Account tab)
- `UserSession.isDarkMode` flag drives all color decisions
- Each fragment has an `applyDarkModeColors()` method that programmatically sets colors
- `DarkModeManager` exposes helper methods: `getBackgroundColor()`, `getCardColor()`, `getTextPrimaryColor()`, `getTextSecondaryColor()`, `getAccentColor()`
- The color palette inverts: beige backgrounds become deep purple, purple accents become warm beige

| Element | Light Mode | Dark Mode |
|---------|-----------|-----------|
| Background | #fce5bd (Beige) | #2D1B2E (Deep Purple) |
| Surface/Card | #FFFFFF (White) | #4A354B (Card Purple) |
| Text Primary | #212121 (Dark Gray) | #F5E6D0 (Warm Beige) |
| Text Secondary | #757575 (Medium Gray) | #C4A882 (Muted Beige) |
| Accent | #87406f (Purple) | #fce5bd (Beige) |

---

## Feature Details

### Social Feed

- Dual-tab interface: **My Feed** and **Messages**
- Post composition with photo attachments from device storage
- Post privacy levels: `PUBLIC`, `CONNECTIONS`, `PRIVATE`
- Like/comment interactions with @mention support
- "Active Members Nearby" carousel using device geolocation
- Account search functionality
- Boosted posts sort to the top of the feed

### Activities & Events

- **Map View**: Google Maps with custom emoji markers per activity type (12 types: Hiking, Surfing, Climbing, Camping, etc.)
- **Calendar View**: Custom grid calendar with month navigation and date filtering
- Activity creation dialog with date/time picker, location, and capacity
- Filter by activity type and date range
- Event detail dialogs with attendee lists and RSVP

### Dating

- Tinder-style swipe card interface with animated gestures
- Profile cards with multi-photo navigation
- Super Like and Undo counters
- Preference-based filtering: gender, interest, intentions, location, age range, search radius
- Safety tips dialog shown during onboarding
- Dating onboarding flow locks tabs until preferences are set
- "Invite Only" badge for exclusive profiles

### Build Guides

- Categorized guide library: Electrical, Insulation, Plumbing, Flooring, Roof, Solar, Furniture
- Free tier: 2 guides available
- Premium tier: Full library access (8 additional guides)
- "Ask a Builder" Q&A submission feature (premium)
- Guide detail pages with step-by-step content and community Q&A

### Account & Profile

- Profile photo and header image upload
- Van icon selector (8 retro/modern van illustrations)
- Activity/interest chips (20 predefined activities)
- Follower/following system
- Settings: dark mode toggle, hide dating profile, manage subscription
- Content reporting system (spam, inappropriate, harassment, fake profile)

---

## RevenueCat Integration

CaraVan uses RevenueCat for subscription management under the product name **"FastTrack"** (entitlement: `"Caravan Pro"`).

### Architecture

```
CareAVanApplication.kt
    └── Purchases.configure()           # SDK initialization on app start

Paywall.kt
    └── PaywallHost (interface)         # Contract for paywall-capable activities
        ├── launchPaywall()             # Opens RevenueCat paywall UI
        └── launchCustomerCenter()      # Opens subscription management

MainActivity.kt
    └── implements PaywallHost
        ├── PaywallActivityLauncher     # Registered before STARTED lifecycle
        └── PaywallResultHandler        # Handles purchase/restore/cancel/error

CheckEntitlement.kt
    └── isPro(callback)                 # Async entitlement check
```

### Initialization

RevenueCat is configured at application startup in `CareAVanApplication.kt`. The SDK is initialized with the project's public API key and debug logging is enabled for development builds.

### PaywallHost Interface

Any activity that needs to present the paywall implements the `PaywallHost` interface:

```kotlin
interface PaywallHost {
    fun launchPaywall()
    fun launchCustomerCenter()
}
```

`MainActivity` is the primary implementer. Fragments access the paywall via:

```kotlin
(activity as? PaywallHost)?.launchPaywall()
```

### Paywall Trigger Points

The paywall is presented when users attempt to access premium content:

| Location | Trigger |
|----------|---------|
| Account Tab | "Upgrade to FastTrack" button |
| Build Help Tab | Tapping a premium-locked guide |
| Guide Detail | Viewing locked Q&A answers |
| Settings Dialog | "Manage Subscription" button |

### Entitlement Checking

Premium status is verified asynchronously using the `CheckEntitlement` utility:

```kotlin
CheckEntitlement.isPro { isPremium ->
    if (isPremium) {
        // Unlock premium content
    } else {
        // Show paywall
    }
}
```

This queries RevenueCat's `getCustomerInfo()` API and checks for the active `"Caravan Pro"` entitlement.

### Purchase Flow

1. User taps premium content or "Upgrade to FastTrack"
2. Fragment calls `(activity as? PaywallHost)?.launchPaywall()`
3. `MainActivity.launchPaywall()` checks current entitlement
4. If not entitled, `PaywallActivityLauncher` opens RevenueCat's pre-built paywall UI
5. User completes purchase through Google Play billing
6. `PaywallResultHandler` receives the result (`Purchased`, `Restored`, `Cancelled`, or `Error`)
7. App updates UI state accordingly

### Premium Features (FastTrack)

| Feature | Free | FastTrack |
|---------|------|-----------|
| Unlimited Swipes | - | Yes |
| See Who Liked You | - | Yes |
| Build Guides | 2 Free | All |
| Ask a Builder | - | Yes |
| Priority Matching | - | Yes |
| Ad-Free Experience | - | Yes |

**Pricing:** $9.99/month, billed through Google Play

---

## Content Assets

Media assets are bundled in the APK under `app/src/main/assets/CaraVan_Content_Assets/`:

```
CaraVan_Content_Assets/
├── Vanlifers/              # Profile photos organized by demographic folders
│   ├── [16 demographic folders with portrait images]
│   └── Each folder contains numbered variants (_1.png, _2.png, etc.)
│
├── Photos_of_Vans/         # Van exterior/interior photography
│   └── [15+ images named by make/model/year]
│
└── Vanlife_Videos/          # Demo video content
```

Assets are loaded at runtime using `AssetManager` and displayed via Glide for images or `MediaPlayer` with `SurfaceView` for video.

---

## Permissions

| Permission | Purpose |
|-----------|---------|
| `INTERNET` | Network access for Firebase, RevenueCat, Maps |
| `ACCESS_FINE_LOCATION` | Precise location for map features and nearby members |
| `ACCESS_COARSE_LOCATION` | Approximate location fallback |
| `READ_MEDIA_IMAGES` | Photo picker for profile/post images (API 33+) |
| `READ_EXTERNAL_STORAGE` | Legacy photo access (API < 33) |

All location access is optional and can be disabled in device settings.

---

## Build Configuration

- **Application ID:** `com.timc.careavan`
- **Orientation:** Portrait-locked on all screens
- **Minification:** Disabled (development phase)
- **ViewBinding:** Enabled
- **DataBinding:** Temporarily disabled
- **Glide annotation processing:** Temporarily disabled for faster builds

---

## Version History

| Date | Version | Highlights |
|------|---------|-----------|
| 2/12/26 | 0.4.1 | Dark mode calendar fixes, dropdown styling, tab corner polish |
| 2/12/26 | 0.4.0 | Video player controls, feed carousel scrolling, FastTrack rename, UI polish |
| 2/11/26 | 0.3.9 | Portrait lock, legal dialogs, Activities tab styling |
| | 0.3.8 | Initial feature-complete on-device build |
| 2/5/26 | 0.3.0 | Running on device. Need to set override for colors |
| 2/5/26 | 0.2.4 | Dating tab onboarding, dating preferences, suggested friends carousel, builder guides paywalled Q&A |
| 2/4/26 | 0.2.3 | App reassessment for hackathon criteria. Replace map pins with activity emojis, add calendar filter, location filters, auto-locate |
| 2/3/26 | 0.1.8 | Dark mode fixes, add road and sun, stylistic consistency, pop-up window and sub-tab selector background colors |
| 2/2/26 | 0.1.5 | Dark mode toggle, hide/delete account, privacy policy, messages tab, Ask a Builder, free vs premium comparison, premium PRO badges, post truncation, request an invite, welcome message |
| 1/30/25 | 0.1.0 | Soft gradient, branding, likes and comments, swipable card stack | Now CaraVan
| 1/23/26 | 0.0.1 | Project start, basic functionality, Care-A-Van
