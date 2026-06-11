# Requirements — Electric Scooter Rental Platform

## What the App Does

A mobile-first web application that lets city commuters find, unlock, and pay for electric scooter rides in real time. Users scan a QR code on any scooter to begin a ride; the app handles authentication, billing, GPS tracking, and ride history. Operators manage the fleet, pricing, and users from an admin dashboard.

---

## Features

### Rider (Customer)

- **Account Registration & Login** — Email/password sign-up + OAuth (Google/Apple)
- **Map View** — Real-time interactive map showing available scooters nearby
- **Scooter Details** — Battery level, estimated range, per-minute rate, distance from user
- **QR Unlock** — Scan QR code on scooter to start a ride
- **Live Ride Tracker** — Running timer, distance covered, and live cost during active ride
- **End Ride** — Lock scooter remotely, auto-generate receipt
- **Ride History** — Past trips with date, duration, distance, and total cost
- **Wallet & Payments** — Add credit/debit card or UPI; auto-charged on ride end
- **Promo Codes** — Apply discount codes at ride start
- **Notifications** — Push alerts for ride start/end, low battery warnings, payment confirmation

### Admin / Operator

- **Fleet Dashboard** — All scooters on map with live status (available / in-use / low-battery / maintenance)
- **Scooter Management** — Add, retire, or update individual scooters; log battery swaps
- **User Management** — View accounts, suspend/reactivate users, resolve disputes
- **Revenue Reports** — Daily/weekly/monthly earnings, ride volume, average duration charts
- **Promo Code Manager** — Create, activate, and expire discount codes
- **Geofence Zones** — Define allowed riding areas and no-park zones on map

---

## User Flow

### Rider Flow

```
1. ONBOARDING
   └─ Download app → Sign Up (name, email, phone, password)
   └─ Verify email → Add payment method → Home screen

2. FIND A SCOOTER
   └─ Home screen shows map with nearby scooter pins
   └─ Tap a pin → View Details (battery %, range, rate/min, walk distance)

3. UNLOCK & RIDE
   └─ Tap "Unlock" → Scan QR code on scooter
   └─ Scooter unlocks (< 3 sec) → Ride begins
   └─ Active Ride screen: live timer, distance, cost counter

4. END RIDE
   └─ Tap "End Ride" → Park in allowed zone
   └─ Scooter locks → Receipt generated (duration, distance, total cost)
   └─ Payment auto-charged to saved method

5. POST-RIDE
   └─ Receipt saved to Ride History
   └─ Option to rate the scooter (1–5 stars)
```

### Admin Flow

```
1. LOGIN → Admin Dashboard
2. View Fleet Map → Filter by status
3. Select Scooter → Update status / view ride history for that unit
4. Reports Tab → Export revenue/ride data as CSV
5. Users Tab → Search user → View rides / suspend account
6. Promo Tab → Create new code → Set discount % or flat amount + expiry
```

---

## Non-Functional Requirements

| Requirement | Detail |
|---|---|
| **Responsiveness** | Mobile-first; fully usable on 375px screens |
| **Unlock Latency** | Scooter unlock confirmation < 3 seconds |
| **GPS Refresh Rate** | Location updated every 5 seconds during active ride |
| **Payment Provider** | Stripe (cards) + Razorpay (UPI) |
| **Uptime SLA** | 99.9% for unlock and payment endpoints |
| **Auth Security** | JWT tokens, refresh rotation, bcrypt password hashing |
| **Data Privacy** | GDPR-compliant; user can request full data deletion |
| **Offline Handling** | Graceful error if network lost mid-ride; cost calculated server-side |
