---
name: american-greetings-corporate-news
description: >-
  Search and page through American Greetings corporate press releases and news items, with
  authors, categories and press imagery resolved.
api: American Greetings Corporate WordPress REST API
base_url: https://corporate.americangreetings.com/wp-json
spec: openapi/american-greetings-corporate-wordpress-rest-openapi.yml
operations:
  - get_wp_v2_posts
  - get_wp_v2_posts_id
  - get_wp_v2_categories
  - get_wp_v2_users
  - get_wp_v2_media
generated: '2026-09-02'
method: generated
source: openapi/american-greetings-corporate-wordpress-rest-openapi.yml
---

# American Greetings corporate news

`/wp/v2/posts` is the machine-readable form of https://corporate.americangreetings.com/latest-news/.
88 items as of 2026-09-02, all readable anonymously. This is the right surface for monitoring
American Greetings announcements; the RSS feed at `/feed/` carries only the most recent items.

## Steps

1. **Page the archive.** `get_wp_v2_posts` — `GET /wp/v2/posts?per_page=100&page=1&_embed`.
   `X-WP-Total` gives the item count and `X-WP-TotalPages` the page count at your `per_page`;
   `per_page` is capped at 100 by the provider's own schema. Follow `Link: rel="next"` to walk.

2. **Filter instead of downloading everything.** The provider's route index declares these query
   parameters on this operation, and the server enforces them:
   - `search=<string>` — full-text.
   - `after=` / `before=` — ISO 8601 publication window.
   - `modified_after=` / `modified_before=` — for incremental sync, which is what you want on a
     repeat run. Store the highest `modified_gmt` you have seen and pass it back as `modified_after`.
   - `categories=<id>` — restrict to a category; get the ids from `get_wp_v2_categories`.
   - `orderby=date&order=desc` — newest first (the default).

3. **Resolve the author.** `post.author` is a User id. `_embed` inlines it under
   `_embedded.author`; otherwise call `get_wp_v2_users` with that id. Anonymous callers get the
   public projection only.

4. **Resolve press imagery.** `post.featured_media` is an Attachment id; `get_wp_v2_media` returns
   `source_url` and `media_details.sizes`. Use the largest size for reproduction.

5. **Fetch one item.** `get_wp_v2_posts_id` — `GET /wp/v2/posts/{id}`. An unknown id returns
   `{"code":"rest_post_invalid_id","data":{"status":404}}`.

## Rules

- `title`, `content` and `excerpt` are `{rendered: "<html>"}` objects. Strip or render the HTML.
- Use `_fields` to cut payload: `_fields=id,date,modified,slug,title,link,categories` is enough for
  a monitoring index and is roughly a tenth the size of the full record.
- Timestamps come in pairs — `date`/`modified` are site-local (GMT-4) and `date_gmt`/`modified_gmt`
  are UTC. Always compare on the `_gmt` pair.
- Read-only. See `conventions/american-greetings-conventions.yml` for the full auth, pagination and
  reversibility picture.
