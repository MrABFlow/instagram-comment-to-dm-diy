# Instagram comment-to-DM starter

A self-hosted n8n + Meta API starter for sending one private reply when a person comments **AUTOMATE** on content owned by your Instagram professional account.

> **Important:** This is a DIY n8n workflow, not a native ManyChat template, ManyChat import, hosted app, or live integration. It has not been tested against a live Meta account. You configure the Meta app, account, HTTPS n8n host, token, and message link yourself.

## Start here

1. Read [SETUP.md](SETUP.md) before importing. It covers Meta permissions, webhooks, n8n configuration, testing, and troubleshooting.
2. Import [comment-to-dm-n8n-starter.json](comment-to-dm-n8n-starter.json) into **self-hosted n8n**. Keep it inactive until configured and tested.

## What happens

Comment **AUTOMATE** → Meta sends the comment webhook → n8n checks the signature and exact keyword → Meta sends one private reply with a public setup link.

The match is case-insensitive and ignores outer whitespace, but extra words do not match. Comments by the connected account are ignored. The JSON points to this repository’s SETUP.md by default; update GUIDE_URL in the Code node if you fork this repository or want to send a different resource.

## What you need

- Instagram Business or Creator account you control, plus a Meta developer app.
- Instagram API with Instagram Login and an Instagram User access token.
- Public HTTPS endpoint on a **self-hosted n8n** instance.
- Your Meta App Secret and a random webhook verification token, stored only on your protected host.
- Node’s built-in crypto module allowed for the Code node.

Meta currently documents instagram_business_basic and instagram_business_manage_comments for this Instagram Login route. Permissions, access levels, app review, and API versions can change; verify them against Meta’s current documentation before launch.

## Security and privacy

The Code node reads META_VERIFY_TOKEN and META_APP_SECRET from process.env and uses Node’s crypto module to validate X-Hub-Signature-256. On n8n 2.x, Code-node environment access is blocked by default. This workflow requires N8N_BLOCK_ENV_ACCESS_IN_NODE=false if used unchanged; that setting exposes environment variables to **all** Code nodes and expressions on the instance. Use it only on a trusted, isolated, single-user host—not a shared or multi-user instance. Otherwise, move signature verification to a separate gateway or use a credential-safe design. See [SECURITY.md](SECURITY.md) and n8n’s environment-security documentation.

The comment payload reaches your n8n host; Meta processes delivery of the reply, and your hosting provider or logs may process or retain data. Do not promise that data never leaves you. The workflow disables its own execution-data saving, but review logs, backups, proxies, and retention separately.

## Meta limits

Meta allows one private reply per comment. For post, Reel, Story, or ad comments, send it within 7 days; for Live comments, reply during the broadcast. A continuing conversation requires the person to respond and must follow Meta’s current messaging policies.

## Files and status

- **comment-to-dm-n8n-starter.json** — inactive importable starter with placeholders; no credentials.
- **SETUP.md** — step-by-step setup and test checklist.
- **SECURITY.md** — secret handling, n8n environment risks, and hardening checklist.
- **.gitignore** — ignores local env files, tokens, and runtime data.

The JSON parses and the workflow structure was reviewed, but it is **not live-tested**. Its example API version is v26.0 as of 2 November 2026; confirm the current supported version before production. Public visibility does not grant a reuse license; no license is included.

## Official references

- [Meta: Private Replies](https://www.postman.com/meta/instagram/request/23987686-189d7215-22b3-403f-b2f5-a46c7e66a514)
- [Meta: Comment Moderation](https://www.postman.com/meta/instagram/folder/23987686-b2d826e8-2090-4859-a3b2-ab1ce4308bc1)
- [Meta: Subscribe to webhooks](https://www.postman.com/meta/instagram/request/23987686-0223707a-7035-46a2-8015-1fdf7249278f)
- [n8n: Code node modules](https://docs.n8n.io/deploy/host-n8n/configure-n8n/basic-configuration/configuration-examples/enable-modules-in-code-node/)
- [n8n: Environment security](https://docs.n8n.io/deploy/host-n8n/configure-n8n/basic-configuration/use-environment-variables/security/)
