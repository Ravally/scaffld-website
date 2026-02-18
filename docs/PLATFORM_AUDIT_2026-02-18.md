# Scaffld Platform Audit — 2026-02-18

## Executive Summary

| Metric | Count |
|--------|-------|
| Total distinct features claimed on website | 41 |
| Features fully built (web app) | 17 |
| Features fully built (mobile app) | 18 |
| Features built on at least one platform | 22 |
| Features partially built (some code, incomplete) | 7 |
| Features not built at all (zero evidence) | 12 |
| **Overall honesty score** | **54%** (22 built / 41 claimed) |

The marketing website promises a polished, all-in-one field service platform with AI, route optimization, GPS fleet tracking, two-way SMS, offline mode, and deep integrations. The reality is a solid quoting-invoicing-job management core with significant gaps in AI (zero AI code exists), SMS (all SMS opens the native messaging app), route optimization (web has none; mobile has basic nearest-neighbor), GPS tracking (mobile only, point-in-time only), and integrations (most listed integrations have no working code behind them).

**Bottom line:** The core workflow — quote → job → invoice → payment — is genuinely well-built on both platforms. But roughly half of what the website advertises either doesn't exist or is dramatically overstated.

---

## Critical Mismatches

Features prominently advertised that DO NOT EXIST in either codebase. Ranked by severity.

### RED — Featured on pricing page or homepage hero. Must build or remove ASAP.

| # | Feature | Where Claimed | Reality |
|---|---------|---------------|---------|
| 1 | **Scaffld AI (all features)** | Features page hero section, pricing page (Grow + Scale tiers), homepage cards | **ZERO AI code in entire codebase.** The "AI rewrite" buttons are 4 lines of string concatenation (`"Great news! " + text`). No LLM API, no OpenAI, no Anthropic, nothing. |
| 2 | **AI Receptionist — answers calls 24/7** | Features page, pricing Scale tier, homepage | **Does not exist.** No telephony, no call handling, no AI. |
| 3 | **AI Voice commands** | Features page, pricing Scale tier | **Does not exist.** No speech recognition code anywhere. |
| 4 | **Route optimization** | Features page, pricing Grow tier, mobile-app page | **Web: Does not exist.** No maps, no routing. **Mobile: Nearest-neighbor heuristic** using straight-line Haversine distance. Not calling Google Directions API. Not real route optimization. |
| 5 | **Two-way SMS & email communication** | Features page (Client Management section) | **One-way only.** All SMS opens the native messaging app — user must tap Send manually. No server-side SMS via Twilio or any provider. No inbound SMS reception. The Twilio settings UI field is completely inert (saves to Firestore, zero code reads it). |
| 6 | **Online credit card & ACH payments (in-app)** | Features page, pricing all tiers | **Partial.** Stripe Checkout links are generated (redirects to Stripe-hosted page). No in-app card collection on web (`@stripe/stripe-js` not installed). Works via redirect, not the seamless experience implied. |
| 7 | **GPS fleet tracking** | Pricing Scale tier, features page, mobile-app page | **Web: Does not exist.** No crew map, no live tracking. **Mobile: Point-in-time GPS stamps** at clock in/out and geofence triggers only. No continuous tracking, no fleet dashboard. |

### YELLOW — Listed on features page. Customers would notice within first week.

| # | Feature | Where Claimed | Reality |
|---|---------|---------------|---------|
| 8 | **Drag-and-drop calendar** | Features page ("Color-coded drag-and-drop calendar") | **Does not exist.** Calendar is custom-built but view-only. No DnD library installed. No drag rescheduling. |
| 9 | **Website Builder** | Features overview grid, pricing Scale tier | **Does not exist.** Zero evidence in any file. |
| 10 | **Revenue heatmap by service area** | Features page, pricing Scale tier | **Does not exist.** Reports have 6 chart types but no geographic heatmap. No map-based revenue visualization. |
| 11 | **QuickBooks Online sync** | Features page, pricing Grow tier, integrations page (marked "Built") | **Partial.** OAuth callback Cloud Function exists. Settings UI has connect buttons. Actual data sync logic is unclear — may work, may not. Cannot confirm functional without testing. |
| 12 | **Card Reader / Tap to Pay** | Features overview grid, mobile-app page, integrations page | **Web: N/A.** **Mobile: Explicit "Coming Soon" placeholder** with disabled button and 50% opacity. `@stripe/stripe-react-native` not installed. Backend endpoints exist but nothing calls them. |
| 13 | **Google Calendar sync** | Integrations page (marked "Available") | **Does not exist.** Connect button renders in settings but zero OAuth flow, zero sync logic behind it. |
| 14 | **Automated crew notifications & dispatch** | Features page | **Partial.** Push notifications work on mobile (job reminders, geofence alerts). No "dispatch notification" that alerts a crew member they've been assigned. Email reminders exist server-side for appointments. |
| 15 | **20+ built-in business reports** | Features page | **Overstated.** There are exactly 6 report types: Revenue, Quote Funnel, Jobs, Invoice Aging, Top Clients, Profitability. Not 20+. |

### GREEN — Mentioned in passing. Nice to have, not a dealbreaker.

| # | Feature | Where Claimed | Reality |
|---|---------|---------------|---------|
| 16 | **Zapier integration** | Integrations page (marked "Available") | Not built. Zero code. |
| 17 | **Mailchimp integration** | Integrations page (marked "Available") | Not built. Zero code. |
| 18 | **Xero sync** | Integrations page (marked "Available") | Similar state to QuickBooks — OAuth callback exists, sync logic unclear. |
| 19 | **Referral system** | Pricing Scale tier ("Marketing Suite: reviews, campaigns, referrals") | Not built. Email campaigns and review requests exist. No referral tracking. |
| 20 | **Smart quote suggestions** | Features page (AI section) | Not built. No pricing intelligence or suggestion engine. |
| 21 | **"Instant payouts to your bank"** | Features page (Invoicing section) | Misleading. Standard Stripe payout schedule applies. Scaffld doesn't control payout timing. |
| 22 | **App available on iOS and Android (App Store / Play Store)** | Mobile app page (download buttons) | **Not published.** The download buttons link to `#`. The app has a bundle ID but is not in any app store. |

---

## Detailed Feature Matrix

### Quoting & Estimates

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Create quotes with line items | "From first quote to final payment" | ✅ Full form with templates, discounts, tax, deposits | ✅ Create with line items, qty, price | No gap |
| Quote PDF generation | Implied by "send quote" | ✅ jsPDF + browser print view | ⬜ Not built | Mobile can't generate/preview PDF |
| Quote email sending | "Send as Email" | ✅ Cloud Function via Resend/SendGrid | 🔶 Sets status to "Sent" but relies on web/backend to dispatch email | |
| Quote SMS sending | Features page | 🔶 Opens native `sms:` link — user must tap Send | 🔶 Same — native SMS link | Not server-side SMS |
| Quote approval (client-facing) | "Client Hub portal" | ✅ Token-based public page, approve/decline, signature | ⬜ Mobile doesn't render public pages | Web handles this |
| Quote → Job conversion | "3 taps" claim | ✅ Firestore trigger auto-creates job on approval | ✅ Manual "Convert to Job" action | |
| Deposit collection on quote | Pricing implies | ✅ Configurable deposit amount/percent, Stripe Checkout | ✅ Via payment link generation | |
| Automated quote follow-ups | Pricing Grow tier | ✅ Cloud Function cron sends after 3 days | N/A (server-side) | |
| Smart quote suggestions (AI) | Features page | ⬜ Not built | ⬜ Not built | 🔴 Claimed, not built |

### Scheduling & Dispatch

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Calendar view | "Scheduling & calendar" | ✅ Month/week/day views, custom built | ✅ List + day-by-staff views | |
| Drag-and-drop scheduling | "Drag-and-drop calendar" | ⬜ Not built — no DnD library | ⬜ Not built | 🔴 Claimed on features page |
| Team assignment | "Team scheduling & dispatch" | ✅ Multi-checkbox staff assignment on jobs | ✅ Via job creation | |
| Recurring jobs | Implied | 🔶 Metadata field only — no recurrence engine generates future jobs | ⬜ | |
| Route optimization | Pricing Grow tier, features page | ⬜ Not built (web) | 🔶 Nearest-neighbor heuristic, Haversine distance — not real Google Directions routing | 🔴 Overstated |
| Automated dispatch notifications | Features page | 🔶 Email appointment reminders exist, no "you've been assigned" push | 🔶 Push notifications for daily job summary, geofence alerts — but no assignment notification | |
| Online booking | Pricing comparison table | ✅ Full 4-step public booking wizard, slot availability | N/A (public web page) | |
| GPS fleet tracking | Pricing Scale tier | ⬜ Not built | 🔶 Point-in-time GPS stamps, geofencing — not continuous fleet tracking | 🔴 "Fleet tracking" is overstated |

### Invoicing & Payments

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Invoice creation | "One-tap invoicing" | ✅ Multi-mode (from job, multi-job, standalone), line items, tax, discount | ✅ Full creation screen | |
| Invoice PDF | Implied | ✅ jsPDF + browser print view | ⬜ No PDF generation on mobile | |
| Invoice email sending | Features page | ✅ Cloud Function via Resend/SendGrid | 🔶 Relies on backend — mobile triggers via status change | |
| Online payments | "Online credit card & ACH payments" | 🔶 Generates Stripe Checkout URL — redirects to Stripe-hosted page. No in-app card form | 🔶 Same — generates link, copy/share/SMS | Not the seamless "Pay Now" button implied |
| Automated payment reminders | Features page, pricing | ✅ Cloud Function cron at 9 AM NZ — due/overdue reminders | N/A (server-side) | |
| Payment plans | Not claimed on website | ✅ Built — installment configuration | Not on mobile | Actually under-advertised |
| Instant payouts | Features page | ⬜ Standard Stripe payout schedule | ⬜ | Misleading claim |
| Tap to Pay / Card Reader | Features overview grid | ⬜ Backend endpoints exist | ⬜ Explicit "Coming Soon" placeholder, SDK not installed | 🟡 Claimed, not built |
| Bulk invoice actions | Not claimed | ✅ Mark paid, archive, delete in bulk | ⬜ | Under-advertised |

### Client / Customer Management

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Client list & detail | "Client management (CRM)" | ✅ Full CRUD, search, filters | ✅ List, detail, creation | |
| Client history (jobs, quotes, invoices) | "Full client history & job records" | ✅ Grouped by status on detail view | 🔶 Navigation to related entities exists | |
| Multiple properties per client | Not explicitly claimed | ✅ Array of properties with addresses | ✅ Referenced in job creation | Under-advertised |
| Client notes | Features page | ✅ Append-only notes feed | ✅ Notes tab on job detail | |
| Client tags | Not claimed | ✅ Multi-tag with bulk tagging | ⬜ | |
| Automated lead tracking & tagging | Features page | 🔶 Tags exist but no auto-tagging logic, no lead scoring | ⬜ | 🟡 "Automated" is overstated |
| Client Hub (self-serve portal) | Features page, pricing all tiers | ✅ Token-based public page: view quotes, jobs, invoices, make payments, submit requests | N/A (web public page) | |
| Two-way SMS communication | Features page | ⬜ One-way only (native `sms:` links) | 🔶 One-way outbound SMS log, no inbound | 🔴 "Two-way" is false |

### Time Tracking

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Crew clock-in/out | "Time Tracking — Crew clock-in/out from the app" | ✅ Clock in/out per job, live timer | ✅ Full stopwatch, GPS stamp on clock in/out, overtime reminder | |
| Timesheet views | Implied | ✅ Filter by staff/job/date, group by date/staff/job, summary cards | 🔶 Today's summary card on clock screen | |
| Payroll export | Not claimed on website | ✅ 5 formats: Xero, MYOB, Standard CSV, Gusto, QuickBooks | ⬜ | Under-advertised |
| GPS on time entries | "GPS time tracking, built in" | ⬜ No GPS capture on web clock in/out | ✅ GPS coordinates captured at clock in and out | Mobile only |
| Automatic mileage logging | Homepage ("automatic mileage logging & route history") | ⬜ Not built | ⬜ Not built — only discrete GPS stamps, no distance tracking | 🟡 Claimed, not built |

### Team Management

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Team invites | Implied by multi-user pricing | 🔶 Settings UI writes staff record, but account linking mechanism unclear | ⬜ | |
| Role permissions (admin/manager/tech) | Implied | ✅ Three roles with route guards + Firestore rules | ✅ Role-aware UI | |
| Staff calendar / availability | "Real-time crew availability" | ⬜ No availability grid | 🔶 Day view groups by staff but no availability concept | 🟡 Claimed, not built |

### Reporting & Analytics

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Business reports | "20+ built-in business reports" | 🔶 6 report types (Revenue, Quote Funnel, Jobs, Invoice Aging, Top Clients, Profitability) | ⬜ No reports on mobile | 🟡 "20+" is false — there are 6 |
| Revenue heatmap | Features page, pricing Scale tier | ⬜ Not built | ⬜ Not built | 🔴 Claimed, not built |
| Dashboard | Homepage mockup shows dashboard | ✅ Today's jobs, schedule, revenue summary, recent activity | ✅ Today's jobs, admin stats | |
| CSV export | Not explicitly claimed | ✅ All 6 reports export to CSV | ⬜ | |
| Job profitability tracking | Features page | ✅ Profitability report (revenue vs labour cost) | ⬜ | |

### AI Features

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| AI Receptionist (24/7 call answering) | Features page hero, pricing Scale | ⬜ **Not built** | ⬜ **Not built** | 🔴 CRITICAL — entire section is fiction |
| AI Voice commands | Features page, pricing Scale | ⬜ **Not built** | ⬜ **Not built** | 🔴 |
| Auto-generated email campaigns | Features page (AI section) | ⬜ Campaign builder exists but is manual, not AI-generated | ⬜ | 🔴 |
| Smart quote suggestions | Features page (AI section) | ⬜ **Not built** | ⬜ **Not built** | 🔴 |
| Scaffld AI (basic) — Grow tier | Pricing page | ⬜ The "AI rewrite" is string concatenation, not AI | ⬜ | 🔴 |

### Mobile App Capabilities

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Mobile app exists | "Mobile app access" on all tiers | N/A | ✅ Full React Native/Expo app | |
| GPS tracking | "Live GPS Tracking" | ⬜ (web) | ✅ Point-in-time GPS + geofencing | |
| Job management | "View schedule, update status, clock in/out" | N/A | ✅ Full job lifecycle | |
| Mobile invoicing | "Create and send invoices on-site" | N/A | ✅ Create invoices + collect via payment link | |
| Photo capture | "Before/after shots auto-attached" | N/A | ✅ Camera + gallery + markup/annotation | |
| On-My-Way texts | "One tap sends ETA" | N/A | ✅ ETA picker + native SMS + Firestore log | |
| Offline mode | "Works without internet, syncs when back" | ⬜ No Firestore persistence enabled on web | ✅ Full AsyncStorage-backed offline queue, auto-sync on reconnect | |
| App Store / Play Store availability | Download buttons on mobile-app page | N/A | ⬜ **Not published** — buttons link to `#` | 🟡 |
| Push notifications | Not explicitly claimed on website | N/A | ✅ Full implementation with deep linking | Under-advertised |
| Geofencing | Not explicitly claimed | N/A | ✅ Auto-alerts on job site arrival/departure | Under-advertised |
| Route optimization (mobile) | Mobile-app page | N/A | 🔶 Nearest-neighbor heuristic with map visualization — not Google Directions | Overstated |

### Integrations

| Integration | Website Status | Actual Status | Gap? |
|-------------|---------------|---------------|------|
| Stripe (Payments) | "Built" | 🔶 Checkout links work. No in-app card collection. Tap to Pay not built. | Partially true |
| QuickBooks | "Built" | 🔶 OAuth callback exists. Sync logic unclear — may work, cannot confirm without testing | Uncertain |
| Google Maps (Routing) | "Built" | 🔶 Places Autocomplete on web. Map + pins on mobile. No Directions API routing. | Overstated |
| Twilio (Communications) | "Built" | ⬜ **Settings UI field only. Zero code calls Twilio API anywhere.** | 🔴 False "Built" label |
| Zapier | "Available" | ⬜ Not built | |
| Google Calendar | "Available" | ⬜ Connect button renders. Zero sync logic. | |
| Mailchimp | "Available" | ⬜ Not built | |
| Xero | "Available" | 🔶 OAuth callback exists, similar state to QuickBooks | |
| Slack | "Coming Soon" | ⬜ Not built | Correctly labeled |
| Square | "Coming Soon" | ⬜ Not built | Correctly labeled |

### Other Features

| Feature | Website Claims | Web App | Mobile App | Gap? |
|---------|---------------|---------|------------|------|
| Job forms & checklists | Pricing Grow tier | ✅ Full form builder, templates, field types, responses | ✅ All field types, photo+signature, offline-aware | No gap |
| Expense tracking | Pricing Grow tier | ✅ Job-scoped, filters, CSV export | ✅ Categories, receipt photos, offline | |
| Email campaigns | Features overview grid | ✅ Campaign builder, segmentation, server-side send, unsubscribe | ⬜ | |
| Review requests | Features overview grid | ✅ Send request, client-facing review page | ⬜ | |
| Email templates | Not claimed on website | ✅ 9 editable templates with variables | N/A | Under-advertised |
| Online booking | Pricing comparison table | ✅ 4-step public wizard, slot availability | N/A | |
| Website builder | Features grid, pricing Scale | ⬜ **Not built** | ⬜ **Not built** | 🔴 |
| Expense tracking (business-level) | "Expense tracking" | 🔶 Job-scoped only — no overhead expenses without a job | 🔶 Same | |

---

## Pricing Tier Audit

### Starter — $39/month

| Feature Listed | Actually Built? | Notes |
|----------------|-----------------|-------|
| 1 user included | ✅ | Single-user default |
| Quoting & invoicing | ✅ | Fully functional on both platforms |
| Scheduling & calendar | ✅ | Calendar exists, no drag-and-drop |
| Client management (CRM) | ✅ | Full CRUD + portal |
| Mobile app access | ✅ | App exists (not in app stores yet) |
| Online payments | 🔶 | Stripe Checkout links, not in-app card collection |
| Client Hub portal | ✅ | Token-based public page |
| ~~Team scheduling~~ | N/A | Correctly excluded from Starter |
| ~~Automated follow-ups~~ | N/A | Correctly excluded from Starter |

**Verdict: 6/7 features built (86%).** Starter tier is the most honest.

### Grow — $79/month (Most Popular)

| Feature Listed | Actually Built? | Notes |
|----------------|-----------------|-------|
| Up to 15 users | 🔶 | Multi-user exists but invite/onboarding flow is unclear |
| Everything in Starter | ✅ | See above |
| Team scheduling & dispatch | 🔶 | Staff assignment works. No dispatch board, no DnD. |
| Route optimization | ⬜ Web / 🔶 Mobile | Web: nothing. Mobile: basic nearest-neighbor heuristic |
| Automated quote follow-ups | ✅ | Cloud Function cron after 3 days |
| Job forms & checklists | ✅ | Full form builder on both platforms |
| QuickBooks Online sync | 🔶 | OAuth exists, sync uncertain |
| Expense tracking | ✅ | Job-scoped, functional |
| Scaffld AI (basic) | ⬜ | **Not built.** "AI rewrite" is string concatenation. |

**Verdict: 5/9 features built (56%).** Route optimization, QuickBooks, and AI are gaps. Over the 30% threshold for concern.

### Scale — $149/month

| Feature Listed | Actually Built? | Notes |
|----------------|-----------------|-------|
| Unlimited users | 🔶 | Same multi-user uncertainty as Grow |
| Everything in Grow | See above | |
| Scaffld AI Receptionist (24/7) | ⬜ | **Not built at all** |
| AI Voice commands | ⬜ | **Not built at all** |
| Marketing Suite (reviews, campaigns, referrals) | 🔶 | Reviews + campaigns built. No referral system. |
| Website builder | ⬜ | **Not built at all** |
| Revenue heatmaps & insights dashboard | ⬜ | 6 reports exist but no heatmaps |
| GPS fleet tracking | ⬜ Web / 🔶 Mobile | No fleet dashboard. Mobile has point-in-time GPS only |
| Priority support | N/A | Operational, not a code feature |

**Verdict: 1/8 features built (13%).** This tier is the most dishonest. Most Scale-exclusive features don't exist. A customer paying $149/mo would find almost none of the differentiating features functional.

---

## Web App — Full Screen Inventory

| View Key | What It Does | Completeness |
|----------|-------------|--------------|
| `dashboard` | Today's jobs, upcoming schedule, revenue summary, recent activity | ✅ Fully functional |
| `clients` | Client list → detail → property detail | ✅ Fully functional |
| `createClient` | New client form with address autocomplete | ✅ Fully functional |
| `quotes` | Quotes list → detail view | ✅ Fully functional |
| `createQuote` | Full quote creation with templates, line items, deposits | ✅ Fully functional |
| `jobs` | Jobs list → detail view with labor/expenses/checklist | ✅ Fully functional |
| `schedule` | List view or calendar (month/week/day) | ✅ Functional (no DnD) |
| `invoices` | Invoice list → detail with payment tracking | ✅ Fully functional |
| `createInvoice` | Multi-mode invoice creation | ✅ Fully functional |
| `timesheets` | Timesheet with filters, grouping, payroll export | ✅ Fully functional |
| `reports` | 6 report tabs with charts and CSV export | ✅ Fully functional |
| `expenses` | Aggregated expenses across all jobs | ✅ Fully functional |
| `settings` | Company, team, integrations, email templates, booking, reviews | ✅ Fully functional |
| `bookings` | Online booking management (approve/decline) | ✅ Fully functional |
| `reviews` | Review list from public submissions | ✅ Fully functional |
| `campaigns` | Email campaign builder, send, detail view | ✅ Fully functional |
| `requests` | Service requests | 🔧 Placeholder ("Coming Soon") |
| `apps` | App marketplace / integrations | 🔧 Placeholder ("Coming Soon") |
| Public: Quote Approval | Client-facing quote approve/decline | ✅ Fully functional |
| Public: Client Portal | View quotes, jobs, invoices, make payments | ✅ Fully functional |
| Public: Booking | 4-step booking wizard | ✅ Fully functional |
| Public: Review | Client review submission form | ✅ Fully functional |
| Public: Unsubscribe | Marketing email opt-out | ✅ Fully functional |

---

## Mobile App — Full Screen Inventory

| Screen | Navigator | What It Does | Completeness |
|--------|-----------|-------------|--------------|
| DashboardScreen | HomeStack (Tab 1) | Today's jobs, stats, admin revenue card | ✅ Fully functional |
| TodayJobsScreen | HomeStack | Today's jobs with list/map toggle | ✅ Fully functional |
| JobDetailScreen | HomeStack, ScheduleStack | Full job detail with Visit/Details/Notes tabs | ✅ Fully functional |
| JobCreateScreen | HomeStack | New job creation form | ✅ Fully functional |
| ClockInOutScreen | HomeStack, MoreStack | Live stopwatch, GPS, today's summary | ✅ Fully functional |
| ClientListScreen | HomeStack, SearchStack | Client list with search | ✅ Fully functional |
| ClientDetailScreen | HomeStack, SearchStack | Client detail view | ✅ Fully functional |
| ClientCreateScreen | HomeStack | New client form | ✅ Fully functional |
| QuotesListScreen | HomeStack, SearchStack | Quotes list with filters | ✅ Fully functional |
| QuoteDetailScreen | HomeStack, SearchStack | Quote detail + status workflow | ✅ Fully functional |
| QuoteCreateScreen | HomeStack | Quote creation with line items | ✅ Fully functional |
| InvoicesListScreen | HomeStack, SearchStack | Invoices list with filters | ✅ Fully functional |
| InvoiceDetailScreen | HomeStack, SearchStack | Invoice detail view | ✅ Fully functional |
| InvoiceCreateScreen | HomeStack | Invoice creation | ✅ Fully functional |
| CollectPaymentScreen | HomeStack | Payment link, SMS, manual recording, Tap to Pay placeholder | 🔶 Tap to Pay is "Coming Soon" |
| RouteDetailScreen | HomeStack | Map + job list with navigation | ✅ Fully functional |
| PhotoMarkupScreen | HomeStack, ScheduleStack | Photo annotation (draw, arrow, text) | ✅ Fully functional |
| JobFormScreen | HomeStack, ScheduleStack | Form renderer with all field types | ✅ Fully functional |
| FormResponseViewScreen | HomeStack, ScheduleStack | Read-only form response display | ✅ Fully functional |
| ExpenseCreateScreen | HomeStack, MoreStack | Expense creation with receipt photo | ✅ Fully functional |
| ScheduleScreen | ScheduleStack (Tab 2) | Date navigation, list/day/map views | 🔶 Map mode is stub |
| SearchScreen | SearchStack (Tab 4) | Cross-entity search with filters | ✅ Fully functional |
| MessageListScreen | MoreStack (Tab 5) | Client message threads | 🔶 Outbound only |
| ConversationScreen | MoreStack | Chat-style conversation view | 🔶 Outbound only, no inbound |
| SettingsScreen | MoreStack | Read-only profile, sign out | 🔶 Minimal — no editing |

---

## Tech Stack Summary

### Web App

| Component | Detail |
|-----------|--------|
| Framework | React 19.1 + Vite |
| Routing | No React Router — state machine (`activeView` string in context) |
| State management | React Context (AuthContext + AppStateContext) |
| Database | Firebase Firestore (multi-tenant: `users/{userId}/`) |
| Auth | Firebase Auth (email/password) |
| Payments | Stripe Checkout (backend generates session URL). No `@stripe/stripe-js` on frontend |
| Email | Resend (primary) or SendGrid (fallback), via Cloud Functions only |
| SMS | Native `sms:` protocol links only — no server-side SMS |
| PDF | jsPDF + jspdf-autotable |
| Charts | Recharts 3.7 |
| Hosting | Firebase Hosting (assumed) |
| Cloud Functions | 10 exports: email sending, Stripe, reminders cron, booking confirmation, review request, campaign send, quote approval trigger |

### Mobile App

| Component | Detail |
|-----------|--------|
| Framework | Expo SDK 54 / React Native 0.81.5 / React 19.1 |
| Navigation | React Navigation 7 (bottom tabs + nested stacks) |
| State management | Zustand 5.0.11 (13 stores) |
| Database | Firebase Firestore (same multi-tenant schema) |
| Auth | Firebase Auth (email/password) |
| Maps | react-native-maps 1.20.1 (Google Maps on Android, Apple Maps on iOS) |
| GPS | expo-location + expo-task-manager (geofencing) |
| Push notifications | expo-notifications (token + deep link routing) |
| Camera | expo-image-picker |
| Offline | Custom AsyncStorage-backed queue + network monitor + auto-sync |
| Haptics | expo-haptics (throughout) |
| Signature | react-native-signature-canvas |
| Photo markup | react-native-webview canvas |
| SMS | Native `sms:` protocol — outbound only |
| PDF | Not available on mobile |
| Stripe Terminal / Tap to Pay | NOT installed (`@stripe/stripe-react-native` absent) |
| Biometrics | NOT installed (`expo-local-authentication` absent) |

---

## Recommendations

### Immediate — Before any public launch or paid marketing

1. **Remove or gate ALL AI claims from the website.** The entire AI section (AI Receptionist, voice commands, smart quotes, auto campaigns) is fiction. Either:
   - Remove AI from the features page and pricing tiers entirely, OR
   - Add "Coming Soon" badges and move AI to a roadmap page

2. **Fix the "Built" label on Twilio integration.** The integrations page labels Twilio as "Built" with a teal badge. Zero code calls Twilio. Change to "Coming Soon" or remove.

3. **Change "Drag-and-drop calendar" to "Calendar scheduling."** The calendar exists but has no drag-and-drop.

4. **Change "20+ business reports" to "6 business reports"** or just say "Built-in business reports."

5. **Change "Two-way SMS & email" to "Email communication + SMS notifications."** There is no inbound SMS.

6. **Change "Route optimization" to "Route planning"** (mobile) and remove from web claims.

7. **Rewrite the Scale tier.** At $149/mo, nearly every differentiating feature (AI Receptionist, voice commands, website builder, revenue heatmaps, GPS fleet tracking) does not exist. This tier cannot be sold as-is.

8. **Remove or disable the app store download buttons** on the mobile-app page until the app is actually published.

### Short-term — Next 2-4 weeks

1. **Implement Stripe Terminal / Tap to Pay** on mobile. The backend endpoints already exist. Install `@stripe/stripe-react-native` and wire up the CollectPaymentScreen placeholder.

2. **Enable Firestore offline persistence** on the web app (`enableIndexedDbPersistence()`). One line of code for a massive improvement.

3. **Implement actual SMS sending** via Twilio Cloud Function. The settings UI already collects the API key — just need the server-side handler.

4. **Install a DnD library** (e.g., `@dnd-kit/core`) and add drag-and-drop to the calendar. The calendar component is already custom-built, so this is achievable.

5. **Verify QuickBooks and Xero sync** actually works end-to-end. If it does, great. If not, change integration labels to "Coming Soon."

6. **Publish the mobile app** to TestFlight and Google Play internal testing.

### Website changes needed

**Features to REMOVE from website (not building soon):**
- AI Receptionist, AI Voice Commands, Smart Quote Suggestions
- Website Builder
- Revenue Heatmaps
- GPS Fleet Tracking (as a dashboard feature)
- "Instant payouts"

**Features to DOWNGRADE on website:**
- "Drag-and-drop calendar" → "Calendar scheduling"
- "20+ reports" → "Business reports"
- "Two-way SMS" → "SMS notifications"
- "Route optimization" → "Route planning"
- Twilio: "Built" → "Coming Soon"
- Google Calendar: "Available" → "Coming Soon"

**Features to ADD to website (built but not advertised):**
- Payment plans (installment billing)
- Payroll export (5 formats)
- Geofencing (auto-alerts on job site arrival)
- Push notifications with deep linking
- Photo markup/annotation tool
- Form builder with 8+ field types including signature
- Email campaign builder with segmentation
- 9 customizable email templates
- Public booking page with slot availability
- Review request system
- Offline mode with sync queue (mobile)
- Multi-property per client
- Sequential invoice numbering
