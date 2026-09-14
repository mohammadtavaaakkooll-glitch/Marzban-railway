<div align="center">

<img src="https://github.com/Gozargah/Marzban-docs/raw/master/screenshots/logo-light.png" width="140" height="140" alt="Marzban Logo">

# Marzban — Railway Edition

### Unified GUI Censorship-Resistant Solution Powered by Xray

**This is the standard Marzban panel, modified only to run on Railway.**

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://hub.docker.com/r/gozargah/marzban)
[![License](https://img.shields.io/badge/License-AGPL--3.0-blue?style=for-the-badge)](https://github.com/Gozargah/Marzban/blob/master/LICENSE)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Xray](https://img.shields.io/badge/Xray-Core-orange?style=for-the-badge)](https://github.com/XTLS/Xray-core)

</div>

---

## Overview

This repository is the **original Marzban panel** with minimal modifications to make it deployable on **Railway**. Nothing has been removed, no features have been stripped, and no core logic has been changed.

The only differences from the upstream Marzban repository are:

- The Dockerfile reads the port from the `$PORT` environment variable injected by Railway
- The startup command is adjusted to bind to `0.0.0.0` and use Railway's assigned port
- Minor environment variable defaults are pre-configured for Railway's PostgreSQL plugin

Everything else — the web UI, the REST API, the Xray integration, the Telegram bot, the CLI, the multi-node support, and every protocol — is **exactly the same as the official Marzban**.

If you know Marzban, you already know this panel. It just runs on Railway now.

---

## Why This Fork Exists

Marzban was designed to be installed on a VPS via a shell script. Railway, however, is a Platform-as-a-Service that:

- Injects the port dynamically via `$PORT`
- Does not provide root access
- Requires environment variables instead of `.env` files
- Uses ephemeral containers unless a Volume is attached

This fork bridges that gap. Nothing more, nothing less.

---

## Deploy on Railway

### Step 1 — Fork and Deploy

1. Fork this repository
2. Go to [railway.com](https://railway.com) and create a new project
3. Choose **Deploy from GitHub repo** and select your fork
4. Wait for the build to complete (approximately 90 seconds)

### Step 2 — Add Database and Storage

| Action | Details |
|---|---|
| Add PostgreSQL | Click `New → Database → PostgreSQL` |
| Add Volume | Settings → Volumes → New Volume → Mount Path: `/var/lib/marzban` |

The Volume is required. Without it, your database, users, and Xray configuration will be erased on every redeploy.

### Step 3 — Set Environment Variables

Go to the **Variables** tab and add the following:

| Variable | Value |
|---|---|
| `SUDO_USERNAME` | `admin` |
| `SUDO_PASSWORD` | your-secure-password |
| `UVICORN_HOST` | `0.0.0.0` |
| `UVICORN_PORT` | `$PORT` |
| `SQLALCHEMY_DATABASE_URL` | `${{Postgres.DATABASE_URL}}` |
| `XRAY_JSON` | `/var/lib/marzban/xray_config.json` |

### Step 4 — Generate Domain

1. Go to **Settings → Networking**
2. Click **Generate Domain**
3. Set the **Target Port** to `8000`
4. Access the dashboard at `https://your-app.up.railway.app/dashboard/`

### Step 5 — Create Admin

Open the **Deploy Logs** tab and wait for `Application startup complete`. Then log in with the credentials you set in the environment variables.

---

## Features

This panel includes every feature of the official Marzban:

- Built-in Web UI
- Fully REST API backend
- Multiple Nodes support for infrastructure distribution and scalability
- Supports VMess, VLESS, Trojan, and Shadowsocks
- Multi-protocol for a single user
- Multi-user on a single inbound
- Multi-inbound on a single port with fallbacks support
- Traffic and expiry date limitations
- Periodic traffic limits (daily, weekly, monthly)
- Subscription link compatible with V2ray, Clash, and ClashMeta
- Automated share link and QR code generation
- System monitoring and traffic statistics
- Customizable Xray configuration
- TLS and REALITY support
- Integrated Telegram Bot
- Integrated Command Line Interface
- Multi-language support
- Multi-admin support (work in progress)

---

## Telegram Bot

The integrated Telegram bot is available exactly as in the official Marzban. To enable it:

1. Create a bot via @BotFather and copy the API token
2. Get your numeric Telegram ID from @userinfobot
3. Add the following variables in Railway:

| Variable | Value |
|---|---|
| `TELEGRAM_API_TOKEN` | `123456:ABC-DEF...` |
| `TELEGRAM_ADMIN_ID` | `123456789` |

4. Redeploy. The bot starts automatically.

---

## Configuration Reference

| Variable | Description | Default |
|---|---|---|
| `SUDO_USERNAME` | Superuser's username | — |
| `SUDO_PASSWORD` | Superuser's password | — |
| `SQLALCHEMY_DATABASE_URL` | Database connection URL | `sqlite:///db.sqlite3` |
| `UVICORN_HOST` | Bind host | `0.0.0.0` |
| `UVICORN_PORT` | Bind port | `8000` |
| `XRAY_JSON` | Path to Xray config | `xray_config.json` |
| `XRAY_EXECUTABLE_PATH` | Path to Xray binary | `/usr/local/bin/xray` |
| `XRAY_ASSETS_PATH` | Path to Xray assets | `/usr/local/share/xray` |
| `XRAY_SUBSCRIPTION_URL_PREFIX` | Prefix for subscription URLs | — |
| `TELEGRAM_API_TOKEN` | Telegram bot token | — |
| `TELEGRAM_ADMIN_ID` | Telegram admin numeric ID | — |
| `JWT_ACCESS_TOKEN_EXPIRE_MINUTES` | Token expiry in minutes | `1440` |
| `DOCS` | Enable `/docs` and `/redoc` | `False` |
| `DEBUG` | Debug mode | `False` |

---

## Local Installation with Docker

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
docker build -t marzban-railway .
docker run -d -p 8000:8000 \
  -v /var/lib/marzban:/var/lib/marzban \
  --name marzban marzban-railway
