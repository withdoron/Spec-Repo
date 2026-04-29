# Event Schema Verification — Community Node

This document lists the **confirmed field names** the backend (Base44 Event entity) uses, based on where events are created/updated and how they are read in Community Node. Use this for Step 3 field mapping in EventEditor.

---

## 1. Where events are created/updated

| Location | What it does |
|----------|--------------|
| **EventsWidget.jsx** | `Event.create(eventData)` on create; `Event.update(id, eventData)` on update. Payload is whatever EventEditor passes to `onSave(eventData)`. |
| **functions/receiveEvent.ts** | Spoke webhook: builds `eventData` and calls `Event.create(eventData)`. This is the only place in the repo that explicitly constructs the full Event payload for create. |

EventsWidget does **not** transform the payload; it passes EventEditor’s object straight to `Event.create` / `Event.update`. So the payload from EventEditor must use the same field names the backend expects.

---

## 2. Backend payload (from receiveEvent.ts)

The Event create payload in `receiveEvent.ts` (lines 85–118) uses these field names:

| Field in payload | Source in webhook | Notes |
|------------------|-------------------|--------|
| **date** | `start_date` | Main start datetime (ISO). Backend uses **date**, not start_date. |
| **end_date** | `end_date` | End datetime (ISO) or null. |
| **duration_minutes** | `duration_minutes` | Number or null. |
| **title** | `title` | |
| **description** | `description` | |
| **status** | `status` | e.g. 'published', 'cancelled'. |
| **location** | `location` | |
| **thumbnail_url** | `images[0]` | Single URL; webhook sets from first image. |
| **pricing_type** | `pricing_type` | e.g. 'free', 'single_price'. |
| **price** | `price` (or 0 if is_free) | Number. |
| **is_pay_what_you_wish** | `is_pay_what_you_wish` | Boolean. |
| **min_price** | `min_price` | Number or null. |
| **event_type** | `event_types[0]` | **Single string** (e.g. 'workshops_classes'). |
| **network** | `networks[0]` | **Single string** (e.g. 'recess') or null. |
| **punch_pass_accepted** | `punch_pass_eligible` | Backend uses **punch_pass_accepted** (Joy Coins; legacy field name). |
| **is_active** | `true` | |
| **business_id** | (not in webhook) | Set by app when creating from Business Dashboard; required for app-created events. |

Webhook also sends (backend may or may not persist on Event):  
`is_all_day`, `is_location_tbd`, `lat`, `lng`, `is_virtual`, `virtual_url`, `virtual_platform`, `ticket_types`, `accepts_rsvps`, `is_recurring`, `recurring_*`, `organizer_name`, `accessibility_features`, `instructor_note` (from `additional_notes`), `age_info`, `capacity`, etc.

---

## 3. How events are read (display/filter)

| File | Fields used on `event` |
|------|------------------------|
| **EventsWidget.jsx** | `event.id`, `event.title`, `event.description`, `event.date`, `event.price`, `event.network`, `event.boost_end_at` |
| **Events.jsx** (filter/list) | `e.date`, `e.title`, `e.description`, `e.location`, `e.price`, `e.event_type`, `e.network`, `e.audience_tags`, `e.setting`, `e.accepts_silver`, `e.wheelchair_accessible`, `e.free_parking`, `e.is_deleted` |
| **EventCard.jsx** | `event.date`, `event.price`, `event.punch_pass_accepted`, `event.status`, `event.title`, `event.thumbnail_url`, `event.event_type`, `event.network`, `event.location` |
| **EventDetailModal.jsx** | `event.date`, `event.price`, `event.punch_pass_accepted`, `event.status`, `event.title`, `event.thumbnail_url`, `event.location`, `event.description`, `event.duration_minutes`, `event.is_virtual`, `event.virtual_platform`, `event.is_location_tbd`, `event.first_visit_free`, `event.ticket_types`, `event.is_pay_what_you_wish`, `event.min_price`, `event.event_type`, `event.network`, `event.is_recurring` |

So the **confirmed field names the backend returns / the app expects** are:

- **date** — main start datetime (not `start_date`)
- **end_date** — end datetime
- **punch_pass_accepted** — boolean (not `punch_pass_eligible`)
- **event_type** — single string (not array)
- **network** — single string (not array)
- **thumbnail_url** — single image URL (display uses this; may be derived from `images[0]` on backend)
- **duration_minutes** — number
- **title**, **description**, **location**, **price**, **status**, **business_id**, **is_active**
- **first_visit_free**, **ticket_types**, **is_pay_what_you_wish**, **min_price** (detail modal)
- **audience_tags**, **setting**, **accepts_silver**, **wheelchair_accessible**, **free_parking** (filters)
- **boost_end_at** (widget)
- **is_deleted** (filter in Events.jsx)

---

## 4. Confirmed mapping for Step 3 (EventEditor → backend)

| EventEditor / Event Node style | Backend (Base44 Event) | Notes |
|--------------------------------|------------------------|--------|
| **start_date** | **date** | Use `date` for main start datetime (ISO). |
| start_date + end_date | date + **end_date** | Keep end_date as-is. |
| punch_pass_eligible | **punch_pass_accepted** | Rename in payload. |
| punch_cost | **punch_cost** | receiveEvent doesn’t set it; if schema has it, send as-is. |
| event_type (single) | **event_type** | Same. |
| networks (array) | **network** (single) | Send first item or null, e.g. `network: formData.networks?.[0] ?? null`. |
| images (array) | **thumbnail_url** and/or **images** | Webhook sets thumbnail_url from images[0]. Send `thumbnail_url: formData.image || null` and optionally `images: formData.image ? [formData.image] : []` if backend stores both. |
| is_free | **is_free** | Same. |
| pricing_type | **pricing_type** | Same. |
| price, min_price, is_pay_what_you_wish | Same | Same. |
| age_info, capacity | **age_info**, **capacity** | Same. |
| status | **status** | Same ('pending_review' / 'published'). |
| business_id | **business_id** | Same (required for app-created events). |

---

## 5. Summary: renames to apply in EventEditor payload

1. **date** — Send start datetime as `date` (not `start_date`).
2. **punch_pass_accepted** — Send Joy Coins flag as `punch_pass_accepted` (not `punch_pass_eligible`).
3. **network** — Send single string: `network: formData.networks?.[0] ?? null` (not array `networks`).
4. **thumbnail_url** — Send main image as `thumbnail_url: formData.image || null`; optionally keep `images: [formData.image]` if backend expects it.

All other core fields (title, description, location, end_date, duration_minutes, pricing_type, price, is_free, is_pay_what_you_wish, min_price, event_type, age_info, capacity, status, business_id, is_active, punch_cost) can be sent with the same names as in the table above.
