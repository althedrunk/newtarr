# NewtArr

A neutered fork of [Huntarr](https://github.com/plexguide/Huntarr.io) v6.6.3, from simpler times, maintained by [ElfHosted](https://store.elfhosted.com), with additional fixes and improvements maintained in this repository.

## Additional changes in this repo

This fork builds on top of ElfHosted's NewtArr with a set of bug fixes and quality-of-life improvements. If you're using TRaSH Guides quality profiles or multiple *arr instances, these changes are particularly relevant.

### Bug fixes

**Radarr: Custom Format scores now respected for upgrade searches** (`v1.1.0`)

The original `get_cutoff_unmet_movies()` function only compared quality rank against the profile cutoff — it completely ignored `cutoffFormatScore`. This meant that movies which had reached the quality cutoff (e.g. Remux-2160p) but were still below the profile's Custom Format score threshold were silently skipped and never searched for upgrades.

This is the default behaviour of TRaSH Guides quality profiles, which set `cutoffFormatScore` to a high value (e.g. 10000) to drive CF-based upgrades. In practice, most of your library would never be upgraded.

**Fix:** The function now reads `cutoffFormatScore` from the quality profile and `customFormatScore` from each movie file. A movie is flagged as cutoff-unmet if either the quality rank *or* the custom format score hasn't been met.

### New features

**Per-instance stats on the dashboard** (`v1.1.0`)

If you have multiple instances of the same app (e.g. `radarr` + `radarr-shared`, `sonarr` + `sonarr-shared`), the dashboard now shows a per-instance breakdown of searches and upgrades triggered, in addition to the combined total.

**Timezone settings** (`v1.1.0`)

General Settings now includes a timezone picker. The selected timezone is applied immediately and persists across container restarts, so the scheduler runs at the correct local time.

### UI fixes

- **Chrome dropdown menus** — dropdowns were rendering white text on a white background in Chrome. Fixed with explicit colour rules on `<select>` and `<option>` elements. (`v1.1.0`)
- **Remove instance button** — the "Remove" button text was overflowing outside the red box. Fixed. (`v1.1.0`)
- **Cache-busting** — all static CSS and JS files now include a version query string (`?v=1.1.x`) so browsers always load the latest files after a container update. (`v1.1.1`)

### Running this fork

```yaml
services:
  newtarr:
    image: ghcr.io/althedrunk/newtarr:latest
    container_name: newtarr
    restart: unless-stopped
    ports:
      - "9705:9705"
    volumes:
      - /opt/media-stack/appdata/newtarr:/config
    environment:
      - TZ=Europe/London
```

The image is built automatically from this repository via GitHub Actions on every push to `main`.

---

## Why this fork?

The original Huntarr project was abandoned under controversial circumstances. The developer introduced telemetry, obfuscated code, and potential security concerns that led to significant community backlash. For context, see [this Reddit thread](https://www.reddit.com/r/selfhosted/comments/1rckopd/huntarr_your_passwords_and_your_entire_arr_stacks/?share_id=uq4GWZe3e0FNKUIXWHiq8).

NewtArr is based on v6.6.3, the last clean release before the controversial changes. It has been customized for use within [ElfHosted](https://store.elfhosted.com), but can be used standalone.

Read the full announcement: [Huntarr Ends Its Hunt, NewtArr Takes It Up](https://store.elfhosted.com/blog/2026/02/24/huntarr-ends-its-hunt-newtarr-takes-it-up/)

## Huntarr Feature Timeline

Understanding why we forked at v6.6.3:

### v5.x — The Simple Original

- 4 apps: Sonarr, Radarr, Lidarr, Readarr
- Core function: Background loop that searches for missing media and quality upgrades
- Single instance per app, simple Flask UI, ~300 lines in background processing
- This is the "hunt the missing stuff" version

### v6.x — Growing Complexity

- Added apps: Whisparr, Eros, Swaparr (stalled download handling)
- Multi-instance support (multiple Sonarr instances, etc.)
- Hourly API cap system, scheduler, hunt history tracking
- Database-backed logging
- `background.py` grew from ~300 to ~717 lines

### v7.x — Scope Explosion (529 commits!)

- Requestarr system introduced — full TMDB discovery, request/approve workflow, multi-user with roles (Owner/Moderator/User)
- Prowlarr integration — indexer management (~26K lines in routes alone)
- Plex OAuth authentication
- Database layer ballooned (348 lines to 108KB)
- Windows service support added

### v8.x — Consolidation

- Cleaned up the 7.x additions, ~9 Python deps
- Still fundamentally an *arr orchestrator

### v9.x — Full Rewrite Into Acquisition Platform

- **NZB Hunt:** Built-in Usenet downloader (NNTP client, yEnc decoder, post-processing) — 228KB of code
- **Tor Hunt:** Built-in BitTorrent client via libtorrent
- **Movie Hunt / TV Hunt:** Internal media libraries bypassing Sonarr/Radarr entirely
- Dependencies doubled (9 to 19+), app code grew from ~22KB to ~480KB
- Transformed from "*arr helper" into "replace your entire stack"

### Why v6.6.3?

If you want just the "hunt missing episodes/movies" functionality, the sweet spot is in the 6.x range:

- **v6.0.x** if you want the absolute minimum (Sonarr/Radarr/Lidarr/Readarr only, no multi-instance)
- **v6.6.3** *(this fork)* if you want multi-instance support + Swaparr but before the Requestarr/Prowlarr bloat

Avoid 7.x+ — that's where the request system, Plex auth, Prowlarr, and the massive DB layer arrived. And 9.x is a completely different application with built-in download clients.

## Changes from upstream Huntarr v6.6.3

- Rebranded to "NewtArr"
- ElfHosted green color scheme
- Authentication disabled by default (designed for SSO-proxied deployments)
- Graceful Docker shutdown (no more hanging on SIGTERM)
- Dead documentation links replaced with tooltips
- Whisparr and Eros app sections un-hidden
- Radarr v5 API compatibility fix
- Upstream CI/telemetry/update-check code removed

## What it does

NewtArr continuously searches your *arr media libraries (Sonarr, Radarr, Lidarr, Readarr, Whisparr) for missing content and items that need quality upgrades. It automatically triggers searches while being gentle on your indexers, helping you gradually complete your media collection.

| Application | Status |
| :---------- | :------------ |
| Sonarr | Supported |
| Radarr | Supported |
| Lidarr | Supported |
| Readarr | Supported |
| Whisparr v2 | Supported |
| Whisparr v3 (Eros) | Supported |

## Running with Docker

```yaml
services:
  newtarr:
    image: ghcr.io/althedrunk/newtarr:latest
    container_name: newtarr
    restart: unless-stopped
    ports:
      - "9705:9705"
    volumes:
      - ./config:/config
    environment:
      - TZ=Europe/London
```

The web UI is available on port 9705.

## Configuration

All configuration is done via the web UI. Settings are stored in `/config/`.

- **Apps**: Configure connections to your *arr instances (URL + API key)
- **Search Settings**: Control how many items to search per cycle, sleep duration, and API rate limits
- **Scheduling**: Set up automated search schedules

## Security

A comprehensive security audit of the inherited v6.6.3 codebase has been performed. See [SECURITY-AUDIT.md](SECURITY-AUDIT.md) for the full report.

**Key findings:** The original codebase contains several security issues (hardcoded secret key, weak password hashing, XSS via innerHTML, no CSRF protection). Most authentication-related issues are mitigated when running behind an SSO proxy (the intended ElfHosted deployment model). Standalone users should review the audit and apply the recommended mitigations.

**Positive:** No telemetry, phone-home code, obfuscated code, or data exfiltration mechanisms were found in the v6.6.3 codebase.

## License

This project is a fork of Huntarr.io. See [LICENSE](LICENSE) for details.
