# Dashboard Content API Contract

Launcher now reads all dynamic news and gallery data from a dedicated Content API.  
Use this document to ship a backend that satisfies the expectations baked into the client.

## Base configuration

| Key | Description | Default |
| --- | --- | --- |
| `content_api_base_url` | Absolute base URL of the Content API. The client appends resource paths such as `dashboard` and `news/{id}`. | `https://content.launcher.juststudio.dev/api/v1/` |
| `content_api_timeout_seconds` | Optional request timeout (seconds). | `12` |
| `content_api_cache_seconds` | Client-side cache lifetime for the dashboard snapshot. | `300` |

> The launcher automatically retries with static fallback data if the API is unreachable, so shipping the API is non-breaking.

## Endpoint: `GET {baseUrl}/dashboard`

Returns the data necessary to populate both the news list and the hero gallery in a single request.

```json
{
  "news": [
    {
      "id": "patch-030",
      "title": "Релиз патча 0.3.0 готов",
      "summary": "Короткое описание используется на карточке и сверху страницы.",
      "publishedAt": "2025-11-12T08:00:00Z",
      "accentColor": "#FFE69839",
      "heroImage": "https://cdn.domain/news/patch-030/cover.jpg"
    }
  ],
  "gallery": [
    {
      "id": "banner-rover",
      "image": "https://cdn.domain/gallery/rover.jpg",
      "title": "WUTHERING WAVES",
      "subtitle": "Герой Rover уже доступен в экспедиции",
      "linkedArticleId": "patch-030"
    }
  ]
}
```

### Field notes

* `news[].id` — required, stable slug; reused to fetch article details and to link gallery slides.
* `news[].summary` — short teaser (displayed under the article title in the reader).
* `news[].publishedAt` — ISO-8601; optional but recommended for localized date pills.
* `news[].accentColor` — optional HEX (`#AARRGGBB`); defaults to brand orange if empty.
* `news[].heroImage` — absolute URL of the hero/banner image.
* `gallery[].linkedArticleId` — required for click-through; omit or `null` to render a non-clickable slide.

## Endpoint: `GET {baseUrl}/news/{id}`

Provides the full article that is shown when a player clicks a news card or gallery slide.

```json
{
  "article": {
    "id": "patch-030",
    "title": "Релиз патча 0.3.0 готов",
    "summary": "Подробности о свежем обновлении, новых ивентах и наградах.",
    "author": "Команда JustRin",
    "publishedAt": "2025-11-12T08:00:00Z",
    "heroImage": "https://cdn.domain/news/patch-030/cover.jpg",
    "externalUrl": "https://juststudio.dev/news/patch-030",
    "body": [
      "Патч 0.3.0 добавляет новые рейды, персонажей и QoL изменения.",
      "Ивент «Алхимический шторм» стартует 15 ноября и продлится две недели.",
      "Зайдите в игру в течение первой недели, чтобы получить бонусную филаментную валюту."
    ]
  }
}
```

### Field notes

* `body` — ordered list of paragraphs. Markdown/HTML is **not** required; send plain text with full sentences.
* `externalUrl` — optional CTA button (“Читать на сайте”). Clients gracefully hide the button when omitted.
* `heroImage` — reused from the list but can point to a more detailed banner for the reader.

## Additional expectations

1. **Caching:** the launcher caches the `/dashboard` payload for `content_api_cache_seconds`. Return HTTP cache headers if server-side caching is also desired.
2. **Errors:** respond with appropriate HTTP status codes; the client treats non-2xx responses as failures and falls back silently.
3. **Localization:** all textual fields must be pre-localized. The client does not translate content.
4. **Image hosting:** images must be reachable over HTTPS and served with CORS headers that allow the launcher to download them directly.
5. **Versioning:** bump the base URL (e.g., `/api/v2/`) if the contract changes; older launchers will continue using the v1 defaults.

With these endpoints implemented, the launcher automatically lights up live news, a real-time gallery, and an in-app article reader with click-through support.
