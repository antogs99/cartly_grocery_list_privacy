

**1. SubscriptionService.swift**

- Add `var hasPlusAccess: Bool { hasSharingAccess }` as a computed alias
- Add `@Published var isTrialEligible: Bool = false`
- In `loadProducts()`, after products load, check `yearlyProduct?.subscription?.introductoryOffer != nil` and set `isTrialEligible` accordingly
- Update `ActivePlan` display strings:
  - `.monthly` → "Cartly Plus · Monthly"
  - `.yearly` → "Cartly Plus · Annual"  
  - `.lifetime` → "Cartly Plus · Lifetime"
  - `.comped` → "Cartly Plus · Active"

**2. PaywallView.swift — full rewrite**

Header: "Unlock Cartly Plus"

Three benefit rows with SF Symbol icons:
- `person.2.fill` — "Share live lists with anyone"
- `link` — "Import recipes from any URL"  
- `mappin.and.ellipse` — "Find aisle locations (Beta)"

Three selectable plan cards (annual preselected):
- Monthly: "$1.99 / month" — no trial mention
- Annual: "14 days free" as badge + "Then $9.99 / year" — preselected, accent border
- Lifetime: show ~~$19.99~~ strikethrough + "$14.99" in large text + "Launch Price · Own it forever" caption

Dynamic CTA button:
- Annual selected + isTrialEligible → "Start 14-Day Free Trial"
- Annual selected + !isTrialEligible → "Start Annual"
- Monthly selected → "Start Monthly"
- Lifetime selected → "Buy Lifetime Access"

Legal footer at bottom:
- "Restore Purchase" tappable link → subscriptionService.restore()
- Renewal text: "Annual plan auto-renews at $9.99/year after free trial. Cancel anytime in your Apple ID settings."
- "Terms of Use" and "Privacy Policy" as tappable links → open URLs via UIApplication.shared.open()
  - Terms URL: // TODO: INSERT_TERMS_URL
  - Privacy URL: // TODO: INSERT_PRIVACY_URL
- All footer text in theme.secondaryText, small font

Background: theme.appBackground — never white or a generic sheet color.

**3. OnboardingView.swift — replace with 4-screen flow**

Keep all existing location permission and store search logic intact — just 
restructure the presentation into 4 screens using a step counter.

Screen 1 — "Build your list fast"
- Large SF Symbol: cart.fill in theme.accent
- Title: "Build your list fast"
- Body: "Add groceries in seconds. Cartly groups them automatically so you shop faster."
- CTA button: "Next" → advance to screen 2

Screen 2 — "Shop smarter with Cartly Plus"
- Title: "Shop smarter with Cartly Plus"
- Three feature rows with SF Symbol icons (theme.accent) and labels:
  - person.2.fill — "Share live lists with anyone"
  - link — "Import recipes from any URL"
  - mappin.and.ellipse — "Find aisle locations (Beta)"
- Subtitle: "Start free — upgrade anytime."
- CTA button: "Next" → advance to screen 3

Screen 3 — existing store picker
- Keep all existing location permission, MapKit search, and store selection logic exactly as-is
- CTA: existing Done/Skip behavior → advance to screen 4

Screen 4 — "Try Cartly Plus Free"
- Header: "Try Cartly Plus Free"
- Subheader: "14 days free, then $9.99/year. Cancel anytime."
- Show all 3 plan cards (same card components as PaywallView — extract into a shared component if practical)
- Dynamic CTA same as PaywallView
- "Maybe Later" text button at bottom → completes onboarding and enters main app
- On purchase success → complete onboarding and enter main app
- Style with GroceryTheme throughout

**4. GroceryItemRow.swift — aisle chip**

Current behavior: chip shown only when preferredAisleStore != nil && authService.user != nil

New behavior:
- preferredAisleStore != nil && guest → show chip with lock.fill icon → tapping shows "Sign in to use aisle lookup" alert with "Sign In" action calling authService.switchToSignIn()
- preferredAisleStore != nil && signed in && !hasPlusAccess → show chip with lock.fill icon → tapping opens PaywallView as sheet
- preferredAisleStore != nil && signed in && hasPlusAccess → existing behavior unchanged

When aisle IS shown (has Plus, aisle value present):
- Add a small "Beta" pill badge next to the aisle value text
- Style: theme.accentSoft background, theme.accent text, caption font, 4pt corner radius

**5. Recipe import button (in GroceryListView.swift composer area)**

Current behavior: guest → sign-in alert; signed in → opens RecipeImportView

New behavior:
- Guest → existing sign-in alert (unchanged)
- Signed in + !hasPlusAccess → open PaywallView as sheet
- Signed in + hasPlusAccess → open RecipeImportView (unchanged)

After sign-in from the recipe gate context: set a flag `pendingRecipeImport = true` on 
AuthService or as local @State. In GroceryListView, watch for authService.user becoming 
non-nil while pendingRecipeImport is true → automatically present RecipeImportView.

**6. SettingsView.swift**

Subscription row update:
- If hasPlusAccess: show activePlan display string ("Cartly Plus · Monthly" etc.) with green dot
- If !hasPlusAccess: show "Upgrade to Cartly Plus" as tappable row → opens PaywallView

**7. GroceryListView.swift**

Share button: already gated on hasSharingAccess — confirm it opens PaywallView when 
!hasPlusAccess. No logic change needed, just verify.

**8. Build**

Run the build command from CLAUDE.md. Confirm success. List any new warnings.
