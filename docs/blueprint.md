# CryptoWatch — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A personal Telegram bot that monitors cryptocurrency prices and notifies users privately when watched coins meet configured alerts. Users manage private watchlists with inline buttons or free-text tickers, set price thresholds and percent-change alerts, request on-demand prices, schedule daily summaries, configure quiet hours, and rely on anti-spam cooldowns. The bot owner receives admin metrics about usage and alert statistics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual Telegram users who want private, low-noise crypto price alerts
- The bot owner (admin) who monitors adoption and alert statistics

## Success criteria

- Users receive accurate price alerts based on their configured thresholds and percent changes
- Admin receives daily metrics about active users and most-fired alerts
- Users can manage their watchlists and alert settings through intuitive inline buttons and commands

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and begin onboarding
- **/price** (command, actor: user, command: /price) — Request on-demand price information for a specific ticker or full watchlist summary
- **Add common coin** (button, actor: user, callback: add_common_coin) — Add a popular cryptocurrency (Bitcoin, Ethereum, Toncoin) to the watchlist
- **Add custom ticker** (button, actor: user, callback: add_custom_ticker) — Prompt user to enter a ticker symbol for a custom cryptocurrency
- **Manage alerts** (button, actor: user, callback: manage_alerts) — Configure price threshold and percent-change alerts for selected coins
- **Configure quiet hours** (button, actor: user, callback: configure_quiet_hours) — Set local time window for suppressing push alerts
- **Set morning summary** (button, actor: user, callback: set_morning_summary) — Enable and schedule a daily local-time summary of watchlist coins
- **/admin_stats** (command, actor: admin, command: /admin_stats) — Send admin metrics to the bot owner

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Request timezone selection
3. Offer tutorial option
4. Display main menu

_Data touched:_ User profile

### Add to watchlist
_Trigger:_ add_common_coin or add_custom_ticker

1. Display coin selection or prompt for ticker
2. Validate ticker symbol
3. Add to watchlist with default alert settings
4. Confirm addition

_Data touched:_ Watchlist entry

### Configure alerts
_Trigger:_ manage_alerts

1. Select coin from watchlist
2. Set price threshold (direction and target)
3. Set percent-change threshold (percent and window)
4. Save and confirm settings

_Data touched:_ Watchlist entry

### Price check
_Trigger:_ /price

1. Parse ticker argument
2. Fetch current price data
3. Display price, change, and alert status
4. Return to main menu

_Data touched:_ Watchlist entry, Alert event

### Morning summary
_Trigger:_ scheduled daily

1. Fetch current prices for all watchlist coins
2. Calculate percent changes
3. Format summary with alert status
4. Send to user at scheduled time

_Data touched:_ Watchlist entry, Alert event

### Quiet hours handling
_Trigger:_ alert event during quiet hours

1. Check if alert occurs during quiet hours
2. Queue alert for later delivery
3. Send summary when quiet hours end

_Data touched:_ Alert event

### Admin metrics
_Trigger:_ /admin_stats

1. Validate admin credentials
2. Fetch and format metrics
3. Send to admin

_Data touched:_ Admin metrics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Telegram user's preferences and settings
  - fields: Telegram ID, display name, timezone, locale, quiet hours start/end, morning summary time, cooldown duration, admin flag
- **Watchlist entry** _(retention: persistent)_ — User's monitored cryptocurrency and alert settings
  - fields: user ID, ticker, display name, enabled alerts, percent-change threshold, percent window, price thresholds, last-known price, last-alert-timestamp
- **Alert event** _(retention: persistent)_ — Record of triggered alerts and delivery status
  - fields: user ID, ticker, alert type, old price, new price, percent change, timestamp, delivered flag
- **Admin metrics** _(retention: persistent)_ — Aggregated usage and alert statistics
  - fields: total users, active users (30d), counts per alert type, top N tickers by alerts fired

## Integrations

- **Telegram** (required) — Bot API messaging, command handling, inline buttons
- **Crypto price API** (required) — Fetch current and historical cryptocurrency prices
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View admin metrics with /admin_stats
- Configure price data source and retry policies
- Adjust default alert cooldown and quiet hours behavior

## Notifications

- Price alerts sent as direct messages
- Morning summaries sent at user-scheduled time
- Admin metrics sent to owner upon request
- Quiet-hour summaries sent after quiet hours end

## Permissions & privacy

- All user data is private and not shared with third parties
- Admin can only view aggregated metrics, not individual user data
- Users can delete their watchlists and account data at any time

## Edge cases

- Price API returns invalid or missing data
- User enters invalid ticker symbol
- Alerts fire during quiet hours
- Multiple alerts for same coin within cooldown window
- User changes timezone or alert settings mid-day

## Required tests

- Verify price alerts trigger at correct thresholds
- Test quiet hours suppression and summary delivery
- Validate morning summary content and timing
- Confirm admin metrics accuracy
- Test error handling for price API failures

## Assumptions

- Users will provide accurate timezone information during onboarding
- Price API will return consistent and reliable data
- Users will understand the difference between price threshold and percent-change alerts
