# Scaffld Platform Audit — Summary
**Date:** 2026-02-18

---

## Executive Summary

| Metric | Count |
|--------|-------|
| Total features claimed on website | 41 |
| Features fully built (at least one platform) | 22 |
| Features partially built | 7 |
| Features not built at all | 12 |
| **Overall honesty score** | **54%** |

The core workflow — quote, schedule, invoice, get paid — is genuinely solid on both web and mobile. But roughly half of what the marketing website promises either doesn't exist or is dramatically overstated.

---

## Top 7 Critical Mismatches (RED)

These are advertised prominently. Customers would expect them on day one.

| # | Claim | Reality |
|---|-------|---------|
| 1 | **Scaffld AI (all features)** — Receptionist, voice commands, smart quotes, auto campaigns | **Zero AI code in entire codebase.** "AI rewrite" buttons are 4 lines of string concatenation. |
| 2 | **AI Receptionist answers calls 24/7** | Does not exist. No telephony code anywhere. |
| 3 | **AI Voice commands** | Does not exist. No speech recognition. |
| 4 | **Route optimization** | Web: none. Mobile: basic nearest-neighbor heuristic (not Google Directions). |
| 5 | **Two-way SMS communication** | All SMS opens native messaging app. User taps Send manually. No server-side SMS. No inbound. Twilio settings field is inert. |
| 6 | **Drag-and-drop calendar** | Calendar exists but is view-only. No DnD library installed. |
| 7 | **GPS fleet tracking** | Web: nothing. Mobile: point-in-time GPS stamps only. No fleet dashboard. |

---

## Pricing Tier Honesty

| Tier | Price | Features Claimed | Features Built | Honesty |
|------|-------|-----------------|----------------|---------|
| **Starter** | $39/mo | 7 | 6 | **86%** — Mostly honest |
| **Grow** | $79/mo | 9 | 5 | **56%** — Route optimization, QuickBooks, AI are gaps |
| **Scale** | $149/mo | 8 | 1 | **13%** — Nearly every differentiating feature is fiction |

The Scale tier at $149/mo is the most problematic. A customer paying for AI Receptionist, website builder, revenue heatmaps, and GPS fleet tracking would find none of these exist.

---

## What's Actually Good (Under-Advertised)

These features ARE built but NOT mentioned on the website:

- Payment plans with installments
- Payroll export (5 formats: Xero, MYOB, CSV, Gusto, QuickBooks)
- Geofencing with auto-alerts on job site arrival/departure
- Push notifications with deep linking to specific records
- Photo markup/annotation tool
- Form builder with 8+ field types including photo and signature
- Email campaign builder with recipient segmentation
- 9 customizable email templates
- Online booking page with slot availability
- Review request system with client-facing page
- Full offline mode with sync queue (mobile)
- Sequential invoice numbering

---

## Immediate Actions

### Remove or fix on website NOW:
1. Remove ALL AI claims (Receptionist, voice commands, smart quotes) — or add "Coming Soon"
2. Change Twilio from "Built" to "Coming Soon" on integrations page
3. Change "Drag-and-drop calendar" → "Calendar scheduling"
4. Change "20+ reports" → "Business reports" (there are 6)
5. Change "Two-way SMS" → "SMS notifications"
6. Rewrite the Scale tier — 87% of its features don't exist
7. Remove app store download buttons until app is published

### Build next (quick wins):
1. Enable Firestore offline persistence on web (1 line of code)
2. Implement server-side SMS via Twilio Cloud Function
3. Install DnD library for calendar drag-and-drop
4. Wire up Stripe Terminal on mobile (backend already exists)
5. Publish mobile app to TestFlight / Play Store internal testing

### Advertise what you've built:
1. Add payment plans to pricing features
2. Add form builder / checklists more prominently
3. Highlight offline mode as a key differentiator
4. Add geofencing to mobile app page
5. Add payroll export to features

---

*Full detailed report: [PLATFORM_AUDIT_2026-02-18.md](PLATFORM_AUDIT_2026-02-18.md)*
