# Edge Cases Reference

Type-specific edge case checklists for Phase 3. Referenced from SKILL.md.

**T1:** Skip or note 3-5 obvious ones.
**T2-T3:** Systematic module scan using the lists below.

---

## Universal Edge Cases (all T2-T3)

- **Registration & accounts** — abandoned flows, duplicates, email conflicts, social login linking
- **Payments** — failed charges, refunds, currency rounding, VAT cross-border, double charges
- **GDPR & security** — data deletion cascade, consent versioning, right to be forgotten, cookie consent
- **Media** — oversized uploads, corrupt files, format validation, storage outages
- **Performance** — image-heavy pages, mobile on slow connection, concurrent operations
- **Email deliverability** — emails in spam, bounced addresses, blacklisted domain, unsubscribe handling
- **Third-party failures** — payment API down, map service unreachable, SMS provider error, CDN outage
- **Migration** — legacy data format mismatches, duplicate detection during import, broken redirect chains
- **Timezone & locale** — scheduled publishing across timezones, date display inconsistencies, currency display

---

## E-shop Edge Cases

- **Cart** — abandoned cart, stock change during checkout, concurrent last-item buyers, cart expiry
- **Checkout** — address validation failure, payment timeout, double submit, partial payment
- **Inventory** — overselling, backorder handling, multi-warehouse sync, reserved stock timeout
- **Shipping** — carrier API down, weight miscalculation, customs declaration, lost package
- **Orders** — status race conditions, partial fulfillment, cancellation after shipment, return after refund
- **Pricing** — overlapping promotions, bulk discount edge cases, coupon stacking, price change during checkout
- **Products** — 1000+ variants performance, out-of-stock display, SEO for deleted/discontinued products

## Booking Edge Cases

- **Double booking** — two users select same last slot simultaneously, race condition
- **No-show** — customer doesn't arrive, fee charging, slot reopening
- **Cancellation cascade** — provider cancels, all affected customers need notification + refund
- **Timezone mismatch** — customer books in their timezone, provider sees different time
- **Overbooking** — intentional overbooking strategy (airlines) vs accidental
- **Recurring conflicts** — recurring booking clashes with one-time booking
- **Buffer violations** — back-to-back bookings with no cleanup time
- **Provider unavailability** — provider calls in sick, all day's bookings need rescheduling
- **Calendar sync conflicts** — external calendar blocks slot that was available in system

## Directory Edge Cases

- **Duplicate listings** — same business listed twice by different users
- **Stale data** — listing hasn't been updated in 2 years, phone number wrong
- **Fake reviews** — business owner posting own reviews, competitor posting negative
- **Claim disputes** — two people claim to own the same business listing
- **Map accuracy** — GPS coordinates wrong, pin on wrong side of street
- **Category misclassification** — listing in wrong category, misleading search results
- **Seasonal businesses** — open only in summer, display during off-season?

## LMS Edge Cases

- **Video not loading** — DRM issues, browser compatibility, bandwidth limitations
- **Quiz attempts** — timer expires mid-quiz, browser crashes, lost answers
- **Certificate fraud** — sharing completion URLs, certificate validity verification
- **Concurrent enrollment** — student in multiple cohorts of same course
- **Content access after expiry** — subscription ends, can student see past completions?
- **Instructor removes content** — students mid-course, lesson disappears
- **Progress calculation** — what counts as "complete"? Watched 90%? Opened page? Passed quiz?

## Marketplace Edge Cases

- **Vendor disappears** — orders pending, buyer paid, vendor inactive
- **Dispute escalation** — buyer and vendor disagree, platform must arbitrate
- **Commission calculation** — discounts, refunds, partial returns affect commission
- **Payout timing** — vendor requests payout during dispute period
- **Review manipulation** — fake reviews, review bombing after dispute
- **Listing hijacking** — unauthorized person editing vendor's listing
- **Price undercutting** — vendor changes price after buyer starts checkout
- **Multi-vendor order** — single cart with items from different vendors, partial shipping

## SaaS Edge Cases

- **Tenant isolation breach** — data from one tenant visible to another (critical security)
- **Plan limit reached** — user tries to add 11th team member on 10-person plan
- **Downgrade data loss** — tenant downgrades, has more data than lower plan allows
- **Trial expiry** — active work in progress, trial ends mid-task
- **Billing failure cascade** — payment fails, grace period, feature restriction, data retention
- **API rate limiting** — tenant exceeds API quota, needs graceful degradation
- **Custom domain SSL** — tenant sets custom domain, SSL certificate provisioning fails
- **Data export volume** — large tenant exports 50GB of data, timeout

## Portal Edge Cases

- **Permission inheritance** — nested groups, conflicting permissions, "deny" vs "not granted"
- **Document versioning** — user edits old version while new version exists
- **SSO session expiry** — corporate SSO token expires during long form submission
- **Announcement fatigue** — too many announcements, users ignore important ones
- **Offline access** — employee in field needs documents without internet
