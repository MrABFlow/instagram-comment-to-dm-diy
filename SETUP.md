# Setup guide

This guide deploys the AUTOMATE comment → one private reply workflow in comment-to-dm-n8n-starter.json. It is for self-hosted n8n and Meta’s Instagram API with Instagram Login. The repository contains no app credentials or access tokens.

> This is **not** a native ManyChat flow or a ManyChat import file. It is a DIY n8n alternative. The JSON will not configure a ManyChat account.

## 1. Confirm prerequisites

Have these ready:

- An Instagram Business or Creator account you control.
- A Meta developer account and app; your account must be an app role/tester during development as Meta requires.
- Self-hosted n8n on a stable public HTTPS domain, with a secure editor login and backups.
- A public HTTPS URL for the resource you want to send. The JSON is prefilled with this repository’s SETUP.md address; if you fork the repo, change GUIDE_URL in the Code node.
- Permission to manage the Meta app and connected Instagram account.

The workflow targets Instagram Login. Meta’s Private Replies documentation currently lists instagram_business_basic and instagram_business_manage_comments. Permission names, access levels, and review requirements may change; verify them in your Meta app and current documentation before launch.

## 2. Configure n8n securely

The workflow reads two server-side values:

- **META_VERIFY_TOKEN** — a long random value you choose and keep private.
- **META_APP_SECRET** — the App Secret from your own Meta app dashboard.

The Code node also imports Node’s built-in crypto module. On self-hosted n8n, allow only crypto with this environment setting:

~~~sh
NODE_FUNCTION_ALLOW_BUILTIN=crypto
~~~

In n8n 2.x, environment access from Code nodes is blocked by default. This starter reads the two values with process.env, so unchanged use requires:

~~~sh
N8N_BLOCK_ENV_ACCESS_IN_NODE=false
~~~

**Security trade-off:** this exposes environment variables to every Code node and expression on that n8n instance. Use it only on a trusted, isolated, single-user host. Do not enable it on a shared or multi-user instance or where untrusted people can edit or run workflows. If that trade-off is unacceptable, move signature verification into a separate trusted gateway or replace this starter with a credential-safe design. See SECURITY.md and n8n’s environment-security documentation.

Put real values only in your protected n8n host environment; never commit a real .env file. If you use external Task Runners, configure the crypto allowlist for the runner environment too. Restart n8n after changing environment settings. Protect the editor and serve it over HTTPS.

## 3. Import and configure the workflow

1. In n8n, choose **Import from file** and select comment-to-dm-n8n-starter.json.
2. Keep the workflow inactive while configuring it. Confirm both Webhook nodes use the path instagram-comment-dm and that the GET and POST triggers are present.
3. Open the **Verify, match keyword, build reply** Code node. GUIDE_URL is prefilled to this repo’s SETUP.md. If you fork the project or want to send another resource, replace it with a public HTTPS URL you control.
4. Confirm API_VERSION is supported by Meta. The example uses v26.0 as of 2 November 2026; verify the live version before production.
5. Open the **Send one private reply** HTTP Request node. Create an n8n **Header Auth** credential with header name Authorization and value Bearer YOUR_OWN_INSTAGRAM_USER_ACCESS_TOKEN. Select it on the node. Enter the token only in n8n’s credential UI—never in code, this repository, or a public issue.

## 4. Configure the Meta app

1. In Meta for Developers, create or open your app and configure Instagram API with **Instagram Login**.
2. Add your professional Instagram account as an app role/tester for development, following Meta’s current instructions.
3. Request only the permissions needed. Meta currently lists instagram_business_basic and instagram_business_manage_comments for this Instagram Login private-reply path. External accounts may require Advanced Access and App Review.
4. Generate an Instagram User access token for the correct professional account. Keep it private and add it to the n8n credential above.
5. In the app’s Webhooks setup, choose the Instagram object and subscribe to the comments field. Use the **Production URL** from the n8n GET Webhook node and enter the same META_VERIFY_TOKEN configured on the host. Meta sends a verification GET request; the workflow returns the challenge only when the token matches.
6. Activate the n8n workflow only after GET verification succeeds and credentials are set.

App-level webhook setup and account-level webhook subscription are separate. Subscribe the professional account to the app’s comments field using Meta’s current Subscribe to webhooks guide. The request shape is:

~~~sh
curl -i -X POST \
  "https://graph.instagram.com/v26.0/$IG_USER_ID/subscribed_apps?subscribed_fields=comments" \
  -H "Authorization: Bearer $IG_ACCESS_TOKEN"
~~~

Set IG_USER_ID and IG_ACCESS_TOKEN in your own secure terminal session; never put real values in this guide or repository. Check Meta’s current API version and request details before running it.

## 5. Test before launch

Use a test post on an account you control. Do not repeatedly message other people.

- Verify the Meta webhook challenge succeeds.
- Comment exactly AUTOMATE; confirm one private reply contains the correct guide link.
- Try automate in lowercase; it should match because comparison trims and ignores case.
- Try a different word or a phrase with extra words; it should not match.
- Comment from the connected creator account; it should be ignored.
- Check the n8n execution and Meta API response. Keep the workflow inactive if signature verification, permissions, or sending fails.

## 6. Understand the limits and privacy

Meta permits one private reply per comment. For post, Reel, Story, and ad comments, send it within 7 days; Live comments can only be answered during the broadcast. The recipient may see it in Inbox or Requests. A continuing conversation requires a response from the recipient and must follow Meta’s messaging policies.

The comment webhook payload reaches your n8n host. Meta processes the outbound reply; your hosting provider and infrastructure may also process or log data. This workflow disables successful/error execution-data saving in its own settings, but review instance logs, proxy logs, backups, and retention separately. Do not claim that data never leaves you.

## Troubleshooting

- **Meta verification fails:** confirm the production URL, GET trigger, META_VERIFY_TOKEN, and HTTPS.
- **403 invalid signature:** check META_APP_SECRET, raw-body handling, and crypto access.
- **No match:** the trimmed comment must equal AUTOMATE, ignoring case; extra words do not pass.
- **No DM:** check account-level comments subscription, token account and scopes, API version, Meta’s response, and the private-reply time limit.
- **Environment-variable error:** check n8n 2.x environment access settings and the security trade-off above; do not weaken a shared instance.

## Official references

- [Meta: Private Replies](https://www.postman.com/meta/instagram/request/23987686-189d7215-22b3-403f-b2f5-a46c7e66a514)
- [Meta: Comment Moderation](https://www.postman.com/meta/instagram/folder/23987686-b2d826e8-2090-4859-a3b2-ab1ce4308bc1)
- [Meta: Subscribe to webhooks](https://www.postman.com/meta/instagram/request/23987686-0223707a-7035-46a2-8015-1fdf7249278f)
- [n8n: Code node modules](https://docs.n8n.io/deploy/host-n8n/configure-n8n/basic-configuration/configuration-examples/enable-modules-in-code-node/)
- [n8n: Environment security](https://docs.n8n.io/deploy/host-n8n/configure-n8n/basic-configuration/use-environment-variables/security/)

Prepared 2 November 2026. Verify all current requirements in your own Meta app before launch.
