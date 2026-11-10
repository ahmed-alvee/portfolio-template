# Portfolio agent

Your CV can now answer questions. This template gives you a portfolio page with an AI agent that talks about your work, day and night.

👉 See a live one: https://ahmed-alvee.github.io/portfolio/

# For humans

## 👋 What is this?

Think of a portfolio page with a small assistant in the corner. A recruiter asks, "What did this person do at their last job?" The agent answers in seconds.

- The page is public. GitHub hosts it at no cost.
- The agent uses only the text that you give it: your CV and your notes.
- The agent refuses all other topics. It does not write poems about cats.

The best part: you do not write code. An AI coding assistant on your laptop does the build. You make the accounts, write about yourself, and approve the steps.

## 🎒 What you need

- A laptop (Mac or Windows).
- Your CV.
- A payment card for $5 of OpenRouter credit. This is the only cost. GitHub and Cloudflare are free.
- One AI coding assistant: Claude Code, Codex, or Antigravity. Antigravity is free.
- Approximately 2 hours, and possibly a coffee. Most of this time is for the text about yourself.

## 🚀 How to use it

Do these steps in the sequence shown. [GUIDE.md](GUIDE.md) gives each step in full, with pictures. [GUIDE.pdf](GUIDE.pdf) is the same guide for print.

1. **Make three accounts:** GitHub, Cloudflare, and OpenRouter. (Guide step 1)
2. **Install one AI assistant** and connect it to Cloudflare. (Guide steps 2 to 4)
3. **Write about yourself.** This is the step that makes your agent good. Put your CV and your notes in a file with the name `prompt.txt`. Keep this file in your Documents folder. Do not put a phone number or a home address in it. (Guide step 5)
4. **Set up OpenRouter.** Add $5 of credit. Set all Zero Data Retention switches to on. Make an API key with a $5 credit limit. (Guide step 6)
5. **Copy this template.** Click **Use this template** at the top of this page. Then click **Create a new repository**. Use the name `portfolio`. Set the visibility to **Public**. (Guide step 7.1)
6. **Start your assistant** in your Documents folder. Paste this text:

   > Clone my GitHub repository called portfolio into this folder. Then read AGENTS.md in it and follow it to build my portfolio. My prompt.txt is in my Documents folder. Use the OpenRouter model openai/gpt-6-luna.

7. **Do what the assistant asks.** It opens a file for your OpenRouter key. Paste the key into the file, then save the file. Do not paste the key into the chat. Read each step before you approve it.
8. **Open your page:** `https://<your-username>.github.io/portfolio/`. Click **Ask my agent**. Ask a question about your work. 🎉

Then send the link to a friend. Let them try to confuse your agent.

## 🛠️ What you can do after the build

The page is yours. Ask your assistant to make a change. For example:

- "Change the headline on my page."
- "Add my new job to the page and to the agent's knowledge."
- "Use a different model."

You can also stop the agent at any time. Delete the key on OpenRouter. The page stays online, and the chat stops. The guide tells you how to remove everything ("Pause or clean up").

## 🔒 Is it safe?

Yes, if you obey two rules: keep the credit limit, and keep private data out of your notes.

- **Your cost has a limit.** The $5 credit limit on your key is the hard stop. Do not remove it.
- **Your key stays secret.** It is on Cloudflare and in one file on your laptop. It does not go to GitHub or to the browser of a visitor.
- **Your notes stay out of GitHub.** Your repository is public, but `prompt.txt` and your CV are not in it.
- **A visitor can ask the agent about all the text that you give it.** Do not give it private data.
- **The model providers do not keep the conversations.** The Worker permits only zero data retention providers.
- **A bot check and a rate limit decrease misuse.** They cannot stop all misuse. The credit limit can.

## 🧯 If something breaks

Do not worry. Paste the full error into your assistant. Ask: "What went wrong and how do I fix it?" The table in [GUIDE.md](GUIDE.md) ("If something breaks") shows the common errors.

---

# For agents

Everything below is for the AI coding assistant, and for people who do the build by hand.

**If you are a coding agent:** read [AGENTS.md](AGENTS.md) first and follow its order of work. It has the ground rules (what must never be committed, where the key goes). The sections below are the technical reference: architecture, manual steps, privacy, security and troubleshooting.

```
Visitor -> GitHub Pages (this page) -> Cloudflare Worker -> OpenRouter -> model (zero data retention providers only)
```

A Cloudflare Worker sits between the page and OpenRouter, so the OpenRouter key never reaches the browser.

## What is in here

```
index.html                     your page (content + chat widget)
app.js                         chat logic, no edits needed
config.js                      Worker address, Turnstile sitekey, greeting (public)
workers/openrouter/            the Worker: worker.js and wrangler.toml
workers/profile.example.txt    template for the agent's rules and knowledge
workers/test-chat.mjs          checks the Worker against the real OpenRouter API
```

**Every id in this repo is a placeholder.** You create your own: Worker address (`<your-subdomain>`), Turnstile sitekey and secret, page origin (`<github-username>`). Ids from the example site belong to another account and do not work for you.

## Manual steps

Your assistant does these for you (see AGENTS.md). By hand:

1. **Copy.** `gh repo create portfolio --template ahmed-alvee/portfolio-template --public --clone`, then `cd portfolio`.
2. **Knowledge.** `cp workers/profile.example.txt workers/profile.txt`, keep the rules, add your name, CV and notes. It is gitignored.
3. **Turnstile.** dash.cloudflare.com > Turnstile > Add widget. Hostname `<github-username>.github.io`, mode Managed. Keep the sitekey and the secret.
4. **Secrets.** In `workers/openrouter/`, create `.dev.vars` (gitignored):
   ```
   OPENROUTER_API_KEY=sk-or-...
   TURNSTILE_SECRET=...
   SIGNING_SECRET=<any long random string, for example from: openssl rand -hex 32>
   ```
5. **Worker.** In `workers/openrouter/wrangler.toml` set `ALLOWED_ORIGIN = "https://<github-username>.github.io"` and `MODEL`. Then:
   ```
   cd workers/openrouter
   npx wrangler login
   npx wrangler deploy --secrets-file .dev.vars
   ```
   A new Cloudflare account asks you to pick a workers.dev subdomain on the first deploy. Keep the printed Worker address.
6. **Page.** In `config.js` set `WORKER_URL`, `TURNSTILE_SITEKEY` and `GREETING`. In `index.html` change everything between the `CHANGE` comment and the chat widget. Keep the ids `ask`, `launch` and `panel`.
7. **Publish.** Commit and push to `main`. Repo Settings > Pages > Source: **GitHub Actions**, then Actions > **Deploy page** > Run workflow. After that, every push to `main` publishes.

## Privacy

- `worker.js` sends `provider: { zdr: true, data_collection: "deny" }` with every request. OpenRouter then routes only to zero data retention providers that do not train on prompts. A model without such a provider fails instead of falling back.
- Also turn on all Zero Data Retention switches, and leave both Data Training switches off, at [Guardrails > Workspace Guardrail > Model & Provider Access](https://openrouter.ai/workspaces/default/guardrails/default/models), then Save. Leave prompt logging off. Account and request settings combine: if either asks for ZDR, ZDR applies.
- OpenRouter keeps request metadata (time, model, token counts), not the text, unless you opt in to prompt logging.
- The Worker logs no visitor text and no IP address.
- `workers/profile.txt` goes to the provider with every question. Keep private details out of it.
- Models with ZDR providers: the **Zero data retention: Supported** filter on [openrouter.ai/models](https://openrouter.ai/models), or `https://openrouter.ai/api/v1/endpoints/zdr`.

## What protects you

- The API key lives only in the Worker (and in your local `.dev.vars`). `config.js` is public, so never put a secret in it.
- Only your page's origin can call the Worker from a browser. Other clients can fake the origin, so this is a speed bump, not a lock.
- Turnstile stops bots. The Worker refuses Cloudflare's test secret unless the page runs on localhost.
- After the bot check the Worker hands out a signed ticket that expires after 2 hours. Without it, no model call happens.
- Rate limits per visitor IP, set in `wrangler.toml`: 5 new chats a minute, 6 messages a minute.
- The Worker accepts only plain text, at most 1,000 characters a message and the last 20 messages.
- The page inserts text with `textContent`, so nothing the agent or a visitor writes can run as HTML. A Content-Security-Policy in `index.html` allows only your own scripts and Turnstile.
- A chat (and its cost) starts only when a visitor opens it.
- Rate limits stop one visitor, not a crowd. The credit limit on your OpenRouter key is what caps cost. Set one.

## When something breaks

| You see | Cause | Fix |
|---|---|---|
| `error: Not allowed` | `ALLOWED_ORIGIN` does not match the page | Exactly `https://<you>.github.io`, then `npx wrangler deploy` |
| `error: Bot check failed` | Turnstile secret and sitekey from different widgets, or hostname missing | Check the widget's hostname and both keys |
| `error: Server misconfigured` | Turnstile test secret on a public page | Put your real Turnstile secret |
| `error: Agent unavailable` | No credit, key limit reached, wrong key, or a `MODEL` with no ZDR provider | `npx wrangler tail` shows OpenRouter's reason. Check balance, key limit and model |
| `error: Too many requests` | Rate limit | Wait a minute |
| `error: Failed to fetch` | Wrong `WORKER_URL` in `config.js`, Worker not deployed, or a brand new workers.dev subdomain | Check `config.js` and the address from `deploy`. A new subdomain can take a few minutes |
| `deploy` fails on `profile.txt` | Step 2 skipped | `cp workers/profile.example.txt workers/profile.txt` |
| Old page after a push | Pages build or browser cache | Wait 2 minutes, hard refresh |

Browser DevTools > Console and Network show the details. `npx wrangler tail` (in `workers/openrouter/`) shows the Worker's logs live.

## Check the Worker

From `workers/`:

```
OPENROUTER_API_KEY=sk-or-... node test-chat.mjs
```

Add `MODEL=<model id>` in front to try another model. It runs the Worker against the real API and spends under a cent.

## Good to know

- Changing `wrangler.toml` or `workers/profile.txt` needs `npx wrangler deploy` again. A deploy keeps the secrets.
- `git push` updates the page only. It never touches the Worker or its secrets.
- To change the model without deploying: dash.cloudflare.com > Workers & Pages > `worker-openrouter` > Settings > Variables and Secrets > edit `MODEL`. Update `wrangler.toml` too, or the next deploy puts the old value back.
- `GUIDE.pdf` is printed from `docs/guide.html`: `google-chrome --headless=new --no-pdf-header-footer --virtual-time-budget=8000 --print-to-pdf=GUIDE.pdf docs/guide.html`. Update both when GUIDE.md changes.
- If you put the Worker on your own domain, add it to `connect-src` in the CSP line of `index.html`.
