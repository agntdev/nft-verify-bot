# NFT Verifier Bot — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that verifies NFT ownership across multiple blockchain networks (default: Ethereum and Polygon), granting verification badges/roles and displaying public NFT metadata. Users prove ownership via signed message challenges or wallet address checks, while admins can configure access controls and notification targets.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group admins
- NFT community members

## Success criteria

- Verification badge assigned to user
- NFT holdings displayed with metadata links
- Admin can view verified user list

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open verification menu with network selection
- **/verify** (command, actor: user, command: /verify) — Initiate NFT verification flow
  - inputs: wallet address or signed challenge
  - outputs: verification status, NFT holdings summary
- **/admin** (command, actor: admin, command: /admin) — Access admin controls for verification records
  - inputs: chat roles, network defaults
  - outputs: verified user list, configuration updates

## Flows

### user_verification
_Trigger:_ /verify or button click

1. Display network selection buttons
2. Request wallet address or challenge signature
3. Validate signature/address
4. Query blockchain for NFT holdings
5. Display verification badge and NFT metadata

_Data touched:_ User, Wallet address, NFT holdings, Verification record

### admin_management
_Trigger:_ /admin

1. Authenticate admin user
2. Display configuration options
3. Update default networks
4. Revoke verifications
5. Set notification channel

_Data touched:_ Verification record, Admin config

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with verification status
  - fields: telegram_user_id, verified_wallet, verification_method, timestamp
- **Wallet address** _(retention: persistent)_ — Blockchain address for NFT ownership check
  - fields: address, network
- **NFT holdings** _(retention: persistent)_ — Cached metadata about user's NFTs
  - fields: collection_name, token_id, metadata_url
- **Verification record** _(retention: persistent)_ — Proof of NFT ownership linkage
  - fields: telegram_user_id, wallet_address, network, timestamp, method
- **Admin config** _(retention: persistent)_ — Bot configuration settings
  - fields: default_networks, notification_target
- **Verification attempt log** _(retention: 90 days)_ — Audit log of verification attempts
  - fields: user_id, attempt_time, success_status

## Integrations

- **Telegram** (required) — Bot API messaging and role management
- **Blockchain indexers** (required) — NFT ownership verification
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set default blockchain networks
- Revoke user verifications
- Configure notification channel/webhook
- View verification logs

## Notifications

- Verification success/failure to user
- Verification event summaries to configured channel

## Permissions & privacy

- Never stores private keys
- Only retains verified wallet addresses
- Verification logs limited to 90 days
- User data only accessible to admins

## Edge cases

- Invalid signature format
- Unsupported blockchain network
- Wallet with no NFT holdings
- Multiple wallets needing verification

## Required tests

- End-to-end verification flow with signature submission
- Admin revocation of verification badge
- Display of cached NFT metadata after network change

## Assumptions

- Default networks are Ethereum and Polygon
- Signature verification is primary method
- Telegram role assignment requires bot admin permissions
- Notification channel is optional but defaults to admin user
