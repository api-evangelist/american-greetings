---
name: american-greetings-brand-and-product-catalog
description: >-
  Read the American Greetings brand portfolio and product-line catalogue from the company's
  corporate WordPress REST API, including the imagery attached to each record.
api: American Greetings Corporate WordPress REST API
base_url: https://corporate.americangreetings.com/wp-json
spec: openapi/american-greetings-corporate-wordpress-rest-openapi.yml
operations:
  - get_wp_v2_brands
  - get_wp_v2_brands_id
  - get_wp_v2_products
  - get_wp_v2_products_id
  - get_wp_v2_media
generated: '2026-09-02'
method: generated
source: openapi/american-greetings-corporate-wordpress-rest-openapi.yml
---

# American Greetings brand and product catalogue

`brands` and `products` are American Greetings' own custom post types — the only two entities on
this API that are the company's rather than the WordPress platform's. Both are readable anonymously.
As of 2026-09-02 each collection held 6 records, so a single page covers the whole catalogue.

## Steps

1. **List the brands.** `get_wp_v2_brands` — `GET /wp/v2/brands?per_page=100&_embed`.
   `_embed` inlines the brand mark under `_embedded["wp:featuredmedia"]`, which saves a second
   round trip to `get_wp_v2_media`. Read `X-WP-Total` to confirm you have them all; if it exceeds
   `per_page`, follow the `Link: rel="next"` header rather than incrementing `page` yourself.

2. **List the product lines.** `get_wp_v2_products` — `GET /wp/v2/products?per_page=100&_embed`.
   Same shape. These map to the public product pages (greeting cards, gift packaging, party
   supplies, stationery, stickers and decals, digital greetings).

3. **Trim the payload.** Add `_fields=id,slug,title,excerpt,link,featured_media` when you only need
   an index. Full records carry rendered HTML in `content` and are large.

4. **Fetch one record.** `get_wp_v2_brands_id` / `get_wp_v2_products_id` —
   `GET /wp/v2/{brands|products}/{id}`. `title`, `content` and `excerpt` are objects with a
   `rendered` key holding HTML, not plain strings; render or strip it, do not print it raw.

5. **Resolve imagery.** If you did not pass `_embed`, `featured_media` is an Attachment id — call
   `get_wp_v2_media` with that id and read `source_url` plus `media_details.sizes`. A
   `featured_media` of `0` means no image is attached.

## Rules

- **Do not use integer ids as durable keys.** They are per-site auto-increment values. `slug` and
  `link` are stable and meaningful; `id` is not portable to any other American Greetings brand site.
- **Read-only.** Every write on this API needs WordPress application-password credentials that
  American Greetings does not issue publicly. Anonymous `POST` returns
  `{"code":"rest_cannot_create", ..., "data":{"status":401}}`.
- **Errors are not RFC 9457.** Branch on the `code` string, never on `message`. See
  `errors/american-greetings-problem-types.yml`.
- **No published rate limit.** No `RateLimit-*` or `Retry-After` header is returned. Stay polite —
  a few requests a second — and back off on any non-2xx.
