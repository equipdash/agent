---
name: equipdash
description: Run a rental or activity business on EquipDash through its MCP server. Use for the day's pickups, returns and sessions, finding and changing bookings, checking what is free on a date, quoting a price, customers, items, payments, refunds and unsigned waivers, whenever the person mentions EquipDash, their shop, their bookings or their customers.
license: MIT
---

# Run an EquipDash shop

EquipDash is booking and rental software for businesses that rent out equipment and run activities,
lessons and classes. Every tool named below comes from the EquipDash MCP server at
`https://app.equipdash.com/mcp/v1`.

If `get_today` is not among your tools, connect the server first:

- Claude Code: `claude mcp add --transport http equipdash https://app.equipdash.com/mcp/v1`, then
  run `/mcp` and sign in.
- Codex: `codex mcp add equipdash --url https://app.equipdash.com/mcp/v1`, then
  `codex mcp login equipdash`.
- Any other MCP client: add a remote (Streamable HTTP) server with that URL. Sign in to EquipDash
  when the client asks, pick the business and click **Allow**. A client that cannot sign in can
  send an API key instead as `Authorization: Bearer <key>` (the key needs the `mcp:access` scope).

## Rules that keep the shop's data right

- **Dates are in the shop's own timezone.** Send dates as `YYYY-MM-DD`, or `YYYY-MM-DD HH:MM:SS`
  when the person named a time. When they named only days, send days: the shop's usual pickup and
  return times are filled in and the result says which times were used. Never convert to UTC.
- **Money is in the shop's currency.** Repeat amounts as the tools return them and never add a
  currency symbol of your own.
- **Ids come from this session.** Get a booking, customer, item or experience id from a list or
  search tool before you use it. A booking's number (for example `691`) goes to `get_booking` as
  `reference`.
- **Preview, ask, then do.** Most tools that change something return a preview first. Show the
  person what will happen (the price, the refund, the email that goes out), wait for a clear yes,
  then call the same tool again with `user_confirmed: true` (or `confirmed: true` on
  `send_payment_link` and `process_refund`). Never confirm on the person's behalf.
- **A refusal is an answer.** An error that names a plan means the business's plan does not
  include that feature; an error about permission means the person signed in cannot do it in
  EquipDash either. Tell them which one it was and do not retry.

## What is on today

1. `get_today` is the day as the Today screen shows it: sessions with booked places and check-ins,
   items to get out by time, rentals to pick up, out now and due back, and anything waiting on the
   owner. Pass `date` for another day and `section: "out_now"` or `section: "people"` to narrow it.
2. `get_needs_you` lists what needs the owner now: booking requests, departures with no guide,
   late returns, unpaid bookings and changes waiting for an OK.
3. `get_late_pickups` and `get_late_returns` give the overdue lists on their own.
4. `get_bookings_missing_waivers` lists bookings with unsigned waivers;
   `send_waiver_reminders` chases them after the person agrees.

## Find a booking

- By customer: `search_customers` with a name, organization, email or phone, then `get_customer`.
- By filter: `list_bookings` with `date_from` / `date_to`, `status`, `booking_type`
  (`rental`, `experience`, `store`), `customer_search` or `unpaid: true`. It returns 10 at a time;
  pass `next_offset` back as `offset` for more.
- One booking in full: `get_booking` with `booking_id` or `reference`.

## Is it free, and what does it cost

- Rentals: `list_products` to find the item, then `check_product_availability` with dates and
  quantity. `calculate_rental_price` prices one or more items over a period with the shop's own
  pricing (seasons, variations, locations included).
- Activities, lessons and classes: `list_experiences`, then `check_experience_availability` with
  `experience_id`, `date` and `participants`. A slot that cannot be booked says why.
- Everything on one date: `list_items_by_date` returns each experience with its open time slots.

Quote the price the tool returned. If a slot or item is full, say so and offer the nearest free
option the tool listed.

## Make or change a booking

1. Find the customer with `search_customers`, or create one with `create_customer`.
2. Call `create_booking` without `user_confirmed` to get the preview: items, times, price and
   anything still missing (participant details, for example). Show it, get a yes, then call again
   with `user_confirmed: true`. `save_as_draft: true` saves it without emailing anyone;
   `confirm_booking` confirms a draft later.
3. New dates: `move_booking_dates` (preview first; the customer and the shop are emailed when it
   goes through). Other edits: `update_booking`. Notes for staff: `add_booking_note`.
4. Cancel: `cancel_booking` without `user_confirmed` returns the cancellation policy and the refund
   that applies. Read it to the person before confirming.

## Payments

- What is left to pay: `get_amount_due`. Every payment and refund on a booking:
  `get_payment_transactions`.
- Online payment: `send_payment_link` emails the customer a link for the balance (or a part of it
  if the person named an amount). Preview first, then `confirmed: true`.
- Cash, bank transfer or a card taken in person: `record_payment` with the amount and method.
  It saves straight away, so check the amount with the person before calling it.
- Refund to the card: `process_refund` with `booking_id` and `amount`, preview first, then
  `confirmed: true`.
- How the business is doing: `get_revenue_summary` with a `period` such as `this_week`, and
  `compare: true` for the period before.

## When you are stuck

`search_help_centre` searches EquipDash's own help articles and returns links. Use it when the
person asks how to set something up, then answer from the article and give them the link.

## Done when

- Every date and amount you report came from a tool result in this session.
- Nothing was created, moved, cancelled, charged, refunded or emailed without the person's yes to
  the preview.
- Any refusal was passed on with its reason (plan or permission), not retried.
