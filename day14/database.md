# Database Schema — Electric Scooter Rental Platform

## Tables

---

### `users`
Stores all registered riders and admin accounts.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| name | VARCHAR(100) | NOT NULL | Full name |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login identifier |
| password_hash | TEXT | NOT NULL | bcrypt hash |
| phone | VARCHAR(20) | NULLABLE | Optional contact |
| role | ENUM | NOT NULL | `rider` / `admin` |
| wallet_balance | DECIMAL(10,2) | DEFAULT 0.00 | Prepaid credits (₹/$) |
| is_active | BOOLEAN | DEFAULT TRUE | FALSE = suspended |
| created_at | TIMESTAMP | DEFAULT NOW() | Registration time |

---

### `scooters`
Represents each physical scooter in the fleet.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| qr_code | VARCHAR(100) | UNIQUE, NOT NULL | Printed on scooter body |
| model | VARCHAR(100) | NOT NULL | e.g. "Segway E2 Pro" |
| battery_pct | SMALLINT | NOT NULL | 0–100 |
| status | ENUM | NOT NULL | `available` / `in_use` / `low_battery` / `maintenance` |
| lat | DECIMAL(9,6) | NOT NULL | Last known latitude |
| lng | DECIMAL(9,6) | NOT NULL | Last known longitude |
| rate_per_min | DECIMAL(5,2) | NOT NULL | Ride cost per minute |
| total_rides | INT | DEFAULT 0 | Lifetime ride count |
| created_at | TIMESTAMP | DEFAULT NOW() | Fleet addition date |

---

### `rides`
One row per ride session; links a rider to a scooter.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| user_id | UUID | **FK → users.id**, NOT NULL | The rider |
| scooter_id | UUID | **FK → scooters.id**, NOT NULL | Scooter used |
| started_at | TIMESTAMP | NOT NULL | Ride begin timestamp |
| ended_at | TIMESTAMP | NULLABLE | NULL while ride is active |
| start_lat | DECIMAL(9,6) | NOT NULL | Pickup coordinates |
| start_lng | DECIMAL(9,6) | NOT NULL | |
| end_lat | DECIMAL(9,6) | NULLABLE | Drop-off coordinates |
| end_lng | DECIMAL(9,6) | NULLABLE | |
| distance_km | DECIMAL(6,2) | NULLABLE | Calculated on ride end |
| duration_min | SMALLINT | NULLABLE | Calculated on ride end |
| total_cost | DECIMAL(8,2) | NULLABLE | NULL until ride ends |
| rating | SMALLINT | NULLABLE | 1–5 stars, post-ride |
| status | ENUM | NOT NULL | `active` / `completed` / `cancelled` |

---

### `payments`
Records every charge attempt linked to a completed ride.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| ride_id | UUID | **FK → rides.id**, NOT NULL | The ride being charged |
| user_id | UUID | **FK → users.id**, NOT NULL | The payer |
| amount | DECIMAL(8,2) | NOT NULL | Amount charged |
| method | ENUM | NOT NULL | `wallet` / `card` / `upi` |
| gateway_ref | VARCHAR(255) | NULLABLE | Stripe / Razorpay reference ID |
| status | ENUM | NOT NULL | `pending` / `success` / `failed` / `refunded` |
| paid_at | TIMESTAMP | NULLABLE | Confirmed payment timestamp |

---

### `promo_codes`
Discount codes created by admins.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| code | VARCHAR(50) | UNIQUE, NOT NULL | e.g. `LAUNCH50` |
| discount_pct | SMALLINT | NULLABLE | Percentage off (or NULL if flat) |
| discount_flat | DECIMAL(6,2) | NULLABLE | Flat amount off (or NULL if pct) |
| max_uses | INT | NOT NULL | Total redemption cap |
| used_count | INT | DEFAULT 0 | Auto-incremented on redemption |
| expires_at | TIMESTAMP | NULLABLE | NULL = never expires |
| is_active | BOOLEAN | DEFAULT TRUE | Admin can deactivate early |

---

### `promo_redemptions`
Junction table — tracks which user applied which promo to which ride.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| promo_id | UUID | **FK → promo_codes.id**, NOT NULL | Promo applied |
| user_id | UUID | **FK → users.id**, NOT NULL | Who redeemed |
| ride_id | UUID | **FK → rides.id**, NOT NULL | On which ride |
| redeemed_at | TIMESTAMP | DEFAULT NOW() | Redemption timestamp |

---

### `geofence_zones`
Admin-defined map zones for riding rules.

| Column | Type | Constraints | Notes |
|---|---|---|---|
| **id** | UUID | **PK**, NOT NULL | Primary Key |
| name | VARCHAR(100) | NOT NULL | e.g. "City Center", "Airport No-Park" |
| zone_type | ENUM | NOT NULL | `allowed` / `no_park` / `restricted` |
| polygon_coords | JSONB | NOT NULL | Array of lat/lng points defining the boundary |
| is_active | BOOLEAN | DEFAULT TRUE | Toggle without deleting |
| created_at | TIMESTAMP | DEFAULT NOW() | |

---

## Entity Relationship Overview

```
users ──────────< rides >────────── scooters
  │                 │
  │                 └──< payments
  │                 └──< promo_redemptions >── promo_codes
  │
  └──< promo_redemptions

geofence_zones (standalone — enforced at ride start/end via lat/lng check)
```

---

## Key Relationships Summary

| Foreign Key | References | Relationship |
|---|---|---|
| rides.user_id | users.id | Many rides → one user |
| rides.scooter_id | scooters.id | Many rides → one scooter |
| payments.ride_id | rides.id | One payment → one ride |
| payments.user_id | users.id | Many payments → one user |
| promo_redemptions.promo_id | promo_codes.id | Many redemptions → one promo |
| promo_redemptions.user_id | users.id | Many redemptions → one user |
| promo_redemptions.ride_id | rides.id | One redemption → one ride |
