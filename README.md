# Reproduction: list search input loses focus (view_configurations)

This is the official Medusa starter (`medusa-starter-default`) on Medusa 2.22.6 with the
`view_configurations` feature flag enabled in `medusa-config.ts`. Nothing else is changed.

## Requirements

Node 20+ and a PostgreSQL server you can create a database on.

## Setup

```bash
cp .env.template .env        # then set DATABASE_URL to an empty database
yarn install
npx medusa db:migrate
yarn seed
npx medusa user -e admin@example.com -p supersecret
yarn dev
```

Open http://localhost:9000/app and log in with the user above. The seed creates one stock
location, four products and 20 inventory items, but no reservations (an empty Reservations
list reproduces the bug too).

## Steps

1. Open Inventory and click the search box.
2. Type two letters, wait about one second, then type one more letter.
3. Repeat on Reservations and on Settings > Locations.
4. Compare with Products, where the same steps work.

## Expected

The search box keeps focus and every letter typed ends up in it.

## Actual

On Inventory, Reservations and Locations, focus moves to the page body after the first
characters and the letters typed afterwards are lost. Products is not affected.

## Cause

The table adapters for these three lists call their list hook without `placeholderData`
(`inventory-table-adapter.tsx`, `reservation-table-adapter.tsx` and `location-table-adapter.tsx`
in `packages/admin/dashboard`), so the list is replaced while the new query loads.
`product-table-adapter.tsx` passes `placeholderData`.
