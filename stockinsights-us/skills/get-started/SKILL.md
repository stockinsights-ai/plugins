---
name: get-started
description: Connect a stockinsights.ai account and begin researching US listed companies after installing the plugin.
---

# Get started with stockinsights.ai US

Explain that a stockinsights.ai account is required. If the connection is not active, ask the user to sign in through the stockinsights.ai prompt shown when they first use a plugin tool. Do not request credentials in chat.

To confirm the connection works, call `resolve_companies` for a well-known ticker such as `AAPL`. If it returns data, setup is working.

Once connected, invite the user to start with a company or research question. Examples:

- How have Apple's revenue and diluted EPS changed over the last three fiscal years?
- What did NVIDIA management say about data-center demand in its latest earnings call?
- Show Boeing's latest 8-K disclosures.

Use the `stock-research` skill for the chosen question. If the tools remain unavailable after sign-in, tell the user to reconnect or re-authorise the stockinsights.ai US connection and restart the host.
