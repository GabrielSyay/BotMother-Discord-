# Bot Mother DC - Complete Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Installation](#installation)
4. [Discord Bot Setup](#discord-bot-setup)
5. [Configuration Files](#configuration-files)
6. [Feature Documentation](#feature-documentation)
7. [Commands Reference](#commands-reference)
8. [Permissions](#permissions)
9. [Troubleshooting](#troubleshooting)
10. [API Integration](#api-integration)

---

## Introduction

Bot Mother DC is a professional-grade Discord-Minecraft integration plugin designed to bridge your gaming community across platforms. It provides comprehensive features including account synchronization, a ticket support system, VoIP management, shop integration with Stripe payments, content moderation, and much more.

### Key Features

- **Account Sync**: Link Minecraft and Discord accounts securely
- **Ticket System**: Professional support ticket management
- **VoIP Management**: Dynamic voice channels with time tracking
- **Shop System**: Stripe-powered payments for VIP positions
- **Content Moderation**: Multi-language profanity filtering and punishment system
- **Chat Bridge**: Bidirectional Minecraft ↔ Discord messaging
- **Staff Roles**: Automatic prefix and permission management
- **CAPTCHA**: Bot prevention for new Discord members

---

## Requirements

### Server Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Minecraft Server | 1.17.x | 1.20.x+ |
| Java Version | 17 | 21 |
| RAM | 2GB | 4GB+ |
| Server Software | Spigot | Paper/Purpur |

### Optional Dependencies

- **Vault**: Required for economy and permissions integration
- **LuckPerms**: Recommended for advanced permission management
- **EssentialsChat**: For chat formatting support
- **TownyChat**: For town-based chat channels

---

## Installation

### Step 1: Download and Install

1. Download `BotMotherDC.jar` from SpigotMC
2. Place the JAR file in your server's `plugins/` folder
3. Start or restart your server
4. The plugin will create its configuration files automatically

### Step 2: Initial Setup

1. Stop your server
2. Navigate to `plugins/BotMotherDC/configs/`
3. Open `config.yml` and configure your Discord bot token (see next section)
4. Configure other settings as needed
5. Start your server

---

## Discord Bot Setup

### Creating a Discord Bot

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Click "New Application" and give it a name
3. Navigate to the "Bot" section
4. Click "Add Bot"
5. Under "Privileged Gateway Intents", enable:
   - Presence Intent
   - Server Members Intent
   - Message Content Intent
6. Copy the bot token

### Bot Permissions

Your bot requires the following permissions (Permission Integer: 8 for Administrator, or specific permissions below):

- Manage Roles
- Manage Channels
- Kick Members
- Ban Members
- View Channels
- Send Messages
- Send Messages in Threads
- Embed Links
- Attach Files
- Read Message History
- Add Reactions
- Use Slash Commands
- Connect (voice)
- Speak (voice)
- Mute Members
- Deafen Members
- Move Members

### Inviting the Bot

1. In Discord Developer Portal, go to OAuth2 → URL Generator
2. Select scopes: `bot`, `applications.commands`
3. Select permissions (Administrator or specific ones above)
4. Copy the generated URL and open it in your browser
5. Select your Discord server and authorize

### Configuration in Plugin

Edit `configs/config.yml`:

```yaml
discord:
  bot-token: "YOUR_BOT_TOKEN_HERE"
  guild-id: "YOUR_SERVER_ID"
```

---

## Configuration Files

### configs/config.yml

Main configuration file with general settings.

```yaml
# Main Settings
debug-mode: false          # Enable detailed logging
first-time-setup: true     # Auto-create Discord infrastructure (set to false after first run!)

# Discord Configuration
discord:
  bot-token: "YOUR_TOKEN"
  guild-id: "YOUR_SERVER_ID"
  
# Feature Toggles
features:
  chat-bridge: true
  sync-system: true
  ticket-system: true
  voip-system: true
  shop-system: true
  captcha: true
  moderation: true
```

### configs/stripe.yml

Stripe payment integration settings.

```yaml
# IMPORTANT: Read the SSL/TLS warning at the top of this file!

stripe:
  enabled: true
  api-key-secret: "sk_test_..."
  webhook-secret: "whsec_..."
  
webhook:
  use-builtin: true         # Use internal webserver
  port: 4242
  cloudflare-tunnel: true   # REQUIRED for production!
```

### configs/positions.yml

VIP positions and staff role configuration.

```yaml
positions:
  vip:
    name: "VIP"
    price: 5.00
    duration: 30            # Days
    discord-role: "VIP_ROLE_ID"
    commands-on-activate:
      - "lp user {player} parent add vip"
    commands-on-expire:
      - "lp user {player} parent remove vip"

staff-roles:
  admin:
    discord-role: "ADMIN_ROLE_ID"
    prefix: "&c[Admin] "
    priority: 100
    commands-on-grant:
      - "lp user {player} parent add admin"
```

### configs/chat.yml

Chat bridge and filtering settings.

```yaml
chat:
  global-channel: "CHANNEL_ID"
  media-channel: "CHANNEL_ID"
  server-channel: "CHANNEL_ID"
  
filters:
  enabled: true
  filter-chat: ["TownyChat", "EssentialsChat"]
  
delay:
  media-seconds: 60
```

### configs/norms.yml

Content moderation rules.

```yaml
moderation:
  enabled: true
  
punishments:
  warning-threshold: 3      # Warnings before mute
  mute-duration: 3600       # Seconds
  mute-threshold: 3         # Mutes before kick
  kick-threshold: 3         # Kicks before ban
  
links:
  whitelist-enabled: true
  whitelist:
    - "youtube.com"
    - "twitch.tv"
  blacklist:
    - "malicious-site.com"
```

---

## Feature Documentation

### Account Synchronization

The sync system links Minecraft accounts to Discord accounts securely.

#### How It Works

1. Player runs `/sync dc` in Minecraft
2. System generates a unique 6-character code
3. Player enters the code in Discord (via button/command)
4. Accounts are linked, rewards are granted

#### Sync Rewards

Configure in `positions.yml`:

```yaml
sync:
  enabled: true
  discord-role: "SYNCED_ROLE_ID"
  commands-on-sync:
    - "lp user {player} permission set botmotherdc.synced"
  commands-on-unsync:
    - "lp user {player} permission unset botmotherdc.synced"
```

---

### Ticket Support System

Professional Discord-based ticket system for player support.

#### Workflow

1. User clicks the "Create Ticket" button
2. Modal form appears for subject and description
3. Private channel is created (only user + staff see it)
4. Staff members receive DM notification with Accept button
5. Staff member accepts, can now respond in channel
6. Optional voice channel can be added for live support
7. Ticket can be closed when resolved

#### Configuration

```yaml
tickets:
  enabled: true
  category-id: "CATEGORY_ID"
  timeout-minutes: 30       # Auto-close unaccepted tickets
  extended-timeout: 1440    # Timeout when no staff (24 hours)
  voice-channels: true      # Enable voice support
```

---

### VoIP Voice Channel System

Dynamic voice channel management with time tracking.

#### Features

- Creates temporary voice channels on demand
- Tracks user voice time
- Screen share unlocked after reaching time threshold
- Auto-cleanup when channels empty
- Configurable user limits

#### Configuration

```yaml
voip:
  enabled: true
  category-id: "CATEGORY_ID"
  user-limit: 10
  alone-timeout-seconds: 60
  screenshare-time-threshold: 3600  # 1 hour of voice time
```

---

### Shop System with Stripe

Secure payment processing for VIP positions.

#### Setup Requirements

1. Create a Stripe account at [stripe.com](https://stripe.com)
2. Get your API keys from Stripe Dashboard → Developers → API keys
3. For production, set up Cloudflare Tunnel or valid SSL certificate

#### Production SSL Setup

**IMPORTANT**: Self-signed certificates do NOT work with Stripe in production!

**Option 1: Cloudflare Tunnel (Recommended - Free)**

1. Install cloudflared: `winget install cloudflare.cloudflared`
2. Login: `cloudflared tunnel login`
3. Create tunnel: `cloudflared tunnel create my-tunnel`
4. Configure and run tunnel
5. Add webhook URL in Stripe: `https://your-tunnel.trycloudflare.com/stripe/webhook`

**Option 2: Let's Encrypt (Free)**

1. Install certbot
2. Obtain certificate: `certbot certonly --standalone -d your.domain.com`
3. Configure plugin to use the certificate files

#### Configuration

```yaml
stripe:
  enabled: true
  api-key-secret: "sk_live_..." # Use live key for production
  webhook-secret: "whsec_..."
  currency: "USD"
  
positions:
  vip:
    name: "VIP"
    price: 5.00
    duration: 30
```

---

### Content Moderation

Comprehensive content filtering and punishment system.

#### Supported Languages

The profanity filter supports multiple languages:
- English (EN)
- Portuguese Brazil (PT-BR)
- Spanish (ES)
- Italian (IT)
- Russian (RU)
- Chinese (ZH)

#### Detection Methods

1. **Direct Match**: Exact word matching
2. **Leetspeak**: Character substitution (a→4, e→3, etc.)
3. **Repetition**: Repeated characters (heeello → hello)
4. **Spacing**: Spaced out words (h e l l o → hello)

#### Punishment Flow

```
Warning 1 → Warning 2 → Warning 3 → Mute (configurable duration)
Mute 1 → Mute 2 → Mute 3 → Kick
Kick 1 → Kick 2 → Kick 3 → Ban
```

All punishments sync between Minecraft and Discord.

---

### Chat Bridge

Bidirectional messaging between Minecraft and Discord.

#### Channels

- **Global**: All messages from all players
- **Media**: Messages with attachments (configurable delay)
- **Server**: Verified players only

#### Configuration

```yaml
chat:
  global-channel: "CHANNEL_ID"
  media-channel: "CHANNEL_ID"
  server-channel: "CHANNEL_ID"
  
  # Support for chat plugins
  filter-chat: ["TownyChat", "EssentialsChat"]
```

---

### Staff Role System

Automatic synchronization of Discord roles to in-game prefixes.

#### Configuration

```yaml
staff-roles:
  admin:
    discord-role: "ROLE_ID"
    prefix: "&c[Admin] "
    chat-prefix: "&c[Admin] &f"
    priority: 100
    commands-on-grant:
      - "lp user {player} parent add admin"
    commands-on-revoke:
      - "lp user {player} parent remove admin"
```

---

### CAPTCHA System

Prevents bot accounts from joining your Discord server.

#### How It Works

1. New member joins Discord
2. System sends CAPTCHA challenge
3. Member must complete verification
4. Upon success, member receives verified role

---

## Commands Reference

### Player Commands

| Command | Description | Permission |
|---------|-------------|------------|
| `/sync dc` | Link Minecraft to Discord | `botmotherdc.sync` |
| `/sync unlink` | Unlink your account | `botmotherdc.sync` |
| `/positions` | View your active positions | `botmotherdc.positions` |

### Admin Commands

| Command | Description | Permission |
|---------|-------------|------------|
| `/botmother reload` | Reload all configurations | `botmotherdc.admin.reload` |
| `/botmother reset` | Reset Discord setup | `botmotherdc.admin.reset` |
| `/positions list` | List all positions | `botmother.admin.positions` |
| `/positions give <player> <pos> <days>` | Grant position to player | `botmother.admin.positions` |
| `/positions remove <player> <pos>` | Remove position from player | `botmother.admin.positions` |
| `/positions check <player>` | Check player's positions | `botmother.admin.positions` |

---

## Permissions

### Base Permissions

| Permission | Description | Default |
|------------|-------------|---------|
| `botmotherdc.sync` | Allow account synchronization | true |
| `botmotherdc.positions` | View own positions | true |
| `botmotherdc.chat` | Use chat bridge | true |

### Admin Permissions

| Permission | Description | Default |
|------------|-------------|---------|
| `botmotherdc.admin.reload` | Reload plugin configurations | op |
| `botmotherdc.admin.reset` | Reset Discord setup | op |
| `botmother.admin.positions` | Manage player positions | op |
| `botmotherdc.admin.*` | All admin permissions | op |

---

## Troubleshooting

### Common Issues

#### Bot Not Coming Online

1. Verify bot token is correct in `config.yml`
2. Check that all Privileged Gateway Intents are enabled
3. Ensure bot has been invited to your server
4. Check console for error messages

#### Sync Not Working

1. Verify `guild-id` is correct
2. Check that sync role exists and bot can manage it
3. Ensure player hasn't already synced
4. Try enabling `debug-mode: true` for more info

#### Stripe Payments Failing

1. **In Test Mode**: Self-signed certificates work
2. **In Live Mode**: You MUST use valid SSL (Cloudflare Tunnel or CA cert)
3. Verify webhook secret is correct
4. Check webhook URL is accessible from internet
5. Enable polling as backup

#### Tickets Not Creating

1. Verify ticket category ID is correct
2. Check bot has Manage Channels permission
3. Ensure button channel exists
4. Check console for permission errors

#### Voice Channels Not Creating

1. Verify VoIP category ID exists
2. Check bot has Voice permissions
3. Ensure category isn't full (Discord limit: 50 channels per category)

### Debug Mode

Enable detailed logging in `config.yml`:

```yaml
debug-mode: true
```

This will output detailed information to console for troubleshooting.

### Log Files

Check `logs/latest.log` for error messages and stack traces.

---

## API Integration

### Data Storage Location

All plugin data is stored in the `plugins/BotMotherDC/data/` folder:

| File | Content |
|------|---------|
| `linked_accounts.yml` | Minecraft UUID ↔ Discord ID mappings |
| `shop_data.yml` | Position purchases and expiration dates |
| `voice_time.yml` | User voice time tracking |
| `tickets.yml` | Ticket history and metadata |
| `moderation_data.yml` | Warnings, mutes, kicks, bans |

### Backup Recommendations

1. Regularly backup the `data/` folder
2. Stop server before editing YAML files manually
3. Use a YAML validator when editing configurations

---

## Support

For assistance:

1. Check this documentation thoroughly
2. Enable debug mode and check logs
3. Visit the SpigotMC discussion page
4. Provide logs and configuration (remove sensitive tokens!)

---

## Credits

**Developer**: BotMother  
**Discord Library**: JDA (Java Discord API)  
**Payment Processing**: Stripe  
**Build System**: Maven

---

**Thank you for using Bot Mother DC!**
