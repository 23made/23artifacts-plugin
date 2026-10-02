# 23artifacts

![The 23artifacts mark](assets/logo.png)

23artifacts publishes what you make with your assistant — a page with its images, video, sound and scripts, a deck, or a single file — at a live address that is private until you share it. You choose exactly who can open it, people leave comments on the page itself, and your assistant reads the comments, makes the changes and answers them.

This plugin gives your assistant three things:

- **The 23artifacts connector**, at `https://mcp.23artifacts.com/mcp`: the tools that publish, find, show and share artifacts, and read and answer their comments. You sign in once with your 23artifacts account.
- **The publish-artifact skill**: how to publish well — your workspace's themes and its real screenshots and logos, access, decks, large sites and the comment loop. Its eleven guides load only when a task needs one.
- **Three commands** where the host runs commands (Claude Code and Cowork): `/23artifacts:publish` publishes a file or folder from disk, `/23artifacts:show` shows an artifact, and `/23artifacts:comments` works through an artifact's comments. In chat they load as skills the assistant uses when they fit.

The same folder installs in ChatGPT, Codex and Claude: `plugin.json` and `mcp.json` are the open Agent Plugins manifest ChatGPT and Codex read, and `.claude-plugin/plugin.json` and `.mcp.json` are Claude's.

## What it runs, sends and fetches

The plugin installs no program, hook or local server, and it holds no key or secret. It adds one remote connector, which talks only to `https://23artifacts.com` over HTTPS. What you ask your assistant to publish — files, names, descriptions — and the comments it writes for you go to your 23artifacts account; what it reads back is your own artifacts, their comments and your workspace's material. The skill and the commands are text your assistant reads. The `/23artifacts:publish` command may ask the connector for a one-use upload address and send a large folder to it from your shell, so the files never pass through the conversation.

[Support](https://23artifacts.com/support) · [Documentation](https://23artifacts.com/docs) · [Privacy policy](https://23artifacts.com/privacy) · [Terms](https://23artifacts.com/terms) · hello@23artifacts.com

## Signing in

The first time the connector is used, or when you choose **Connect**, your browser opens 23artifacts.com. Sign in with your email and password, with a link sent to your email, or with Apple — or create an account there. Then an **Authorize access** page names the assistant that is asking — for example "ChatGPT (chatgpt.com) wants to access your 23artifacts account as you@example.com" — lists what it gets (confirm your identity, your name and profile info, your email address, stay connected while you're away), and offers **Allow** and **Deny**. Allow returns you to the assistant, connected. Its access renews itself while you use it and lapses after seven days unused.

## Install

### Claude Code

Install it — this repository is its own marketplace:

```sh
claude plugin marketplace add 23made/23artifacts-plugin
claude plugin install 23artifacts@23artifacts
```

Or try it for one session from a clone of this repository:

```sh
claude --plugin-dir /path/to/23artifacts-plugin
```

In a session, `/mcp` → `plugin:23artifacts:23artifacts` → **Authenticate** signs you in. The commands are `/23artifacts:publish`, `/23artifacts:show` and `/23artifacts:comments`.

Uninstall: `claude plugin uninstall 23artifacts@23artifacts`, then `claude plugin marketplace remove 23artifacts`.

### Claude — claude.ai, the desktop app and Cowork

Plugins come with every paid plan (Pro, Max, Team and Enterprise).

1. **Customize → Plugins → Add → Add marketplace**, and enter `https://github.com/23made/23artifacts-plugin` (or `23made/23artifacts-plugin`).
2. In the Plugins tab's **Discover** list, choose 23artifacts and **Add**.
3. Open the plugin's **Connectors** tab and **Connect** 23artifacts; sign in as above. On a Team or Enterprise plan, an Owner adds the connector for the organization first.

The marketplace keeps the plugin up to date. To install from a file instead, zip a clone of this repository from inside it — `zip -r ../23artifacts-plugin.zip . -x '.git/*'` — and choose **Customize → Plugins → Add → Upload plugin**.

Without the plugin: **Customize → Connectors → Add custom connector**, URL `https://mcp.23artifacts.com/mcp`, then **Connect**; and for the skill, zip the `skills/publish-artifact` folder from inside `skills/` (so the archive's top level is `publish-artifact/`) and upload it under **Customize → Skills**.

Uninstall: **Customize → Plugins → 23artifacts → Remove**, and remove the marketplace from its **⋯** menu. A connector added on its own is removed under **Customize → Connectors**.

### Codex

Add this repository as a marketplace and install the plugin, then sign in:

```sh
codex plugin marketplace add 23made/23artifacts-plugin
codex plugin add 23artifacts@23artifacts
codex mcp login 23artifacts
```

A local copy works too: `codex plugin marketplace add /path/to/23artifacts-plugin`. `codex mcp list` shows the connector, and the skill appears as `23artifacts:publish-artifact`.

The connector alone, without the skill:

```sh
codex mcp add 23artifacts --url https://mcp.23artifacts.com/mcp
codex mcp login 23artifacts
```

Keep one or the other: both use the name `23artifacts`.

Uninstall: `codex plugin remove 23artifacts@23artifacts` and `codex plugin marketplace remove 23artifacts`; or, for the connector alone, `codex mcp remove 23artifacts`.

### ChatGPT

Until 23artifacts is in ChatGPT's Plugins Directory:

1. **Settings → Security and login → Developer mode**, on.
2. Open [ChatGPT Plugins](https://chatgpt.com/plugins), select **+**, name it 23artifacts, enter `https://mcp.23artifacts.com/mcp` as the MCP server URL with OAuth, and create it. Sign in as above. Its tools are then available in your chats.
3. For the skill as well, in the ChatGPT desktop app: add this repository as a marketplace with the `codex plugin marketplace add` line above (the app reads the marketplaces Codex is configured with), restart the app, open the Plugins Directory, choose the 23artifacts marketplace and install the plugin.

A workspace admin can then share it inside the workspace: [ChatGPT Plugins](https://chatgpt.com/plugins) → **Personal** → the plugin's **⋯** → **Publish**, and choose the roles that get it. That never lists it publicly.

Uninstall: remove 23artifacts under [ChatGPT Plugins](https://chatgpt.com/plugins); in the desktop app, uninstall it from the Plugins Directory and remove the marketplace with `codex plugin marketplace remove 23artifacts`.

## Problems and feedback

Something not working, or not working the way you expected? [Open an issue](https://github.com/23made/23artifacts-plugin/issues/new/choose) in this repository: say which assistant you use, the plugin's version, what you asked and what happened. For your account, billing or anything private, use [23artifacts.com/support](https://23artifacts.com/support). Report a security problem privately to security@23artifacts.com, never in an issue.

Or tell your assistant — it can send feedback to 23artifacts directly, with `send_feedback`, and answers with a reference you can quote.

This repository is generated from the 23artifacts source and published automatically, so pull requests here can't be merged — [CONTRIBUTING.md](CONTRIBUTING.md) says how to suggest a change.
