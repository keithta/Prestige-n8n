# n8n template

This template deploys a self-hosted version of [n8n](https://n8n.io/). Internally it uses a PostgreSQL database to store the data.

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/template/EfkjX2?referralCode=lJoDnn)

## ✨ Features

- n8n
- PostgreSQL

## 💁‍♀️ How to use

- Click the Railway button 👆
- Add the required environment variables
- Deploy

## 📝 Notes

- Source image: https://hub.docker.com/r/n8nio/n8n
- Docs: https://docs.n8n.io/

## Campaign workflows

This deployment also hosts the automation for the Microsoft Graph email
campaign application in
[`keithta/Prestige-MCP`](https://github.com/keithta/Prestige-MCP).

The workflow exports live in [`workflows/`](workflows/), with setup notes in
[`docs/N8N-SETUP.md`](docs/N8N-SETUP.md).

They wake the sending worker, deliver alerts, and email a daily digest. They
have **no authority to send email**: the database role they use cannot read
contacts, cannot touch the send queue, and cannot approve or start a campaign.
Deleting them would not stop or change any campaign.
