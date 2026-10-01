# jjs-joints

Shopify theme for **JJ's Joints**, a musician lifestyle brand specializing in guitar cases.

This repo is the source of truth for the online store theme. It's developed locally with the [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) and deployed via Shopify's [GitHub integration](https://shopify.dev/docs/storefronts/themes/tools/github).

## Stack

- **Shopify Online Store 2.0** theme (Liquid, JSON templates, sections & blocks)
- **Shopify CLI** for local development, previewing, and theme checks
- **GitHub integration** to sync branches with themes in the Shopify admin

## Prerequisites

- [Node.js](https://nodejs.org/) (LTS)
- [Git](https://git-scm.com/)
- Shopify CLI:

  ```sh
  npm install -g @shopify/cli@latest
  ```

- Staff or collaborator access to the JJ's Joints Shopify store

## Local development

```sh
git clone <repo-url>
cd jjs-joints

# Start a local dev server with hot reload against the store
shopify theme dev -e development
```

The CLI prints a local preview URL (usually `http://127.0.0.1:9292`) and a shareable preview link. Changes to Liquid, CSS, and JS files reload automatically.

### Environments (`shopify.theme.toml`)

Store and theme settings for the CLI live in [`shopify.theme.toml`](shopify.theme.toml) at the repo root, so you don't have to pass `--store` and `--theme` flags on every command. Each `[environments.<name>]` block defines a target, selected with `-e <name>` (or `--environment <name>`):

```toml
[environments.development]
store = "<store-name>.myshopify.com"

[environments.testing]
store = "<store-name>.myshopify.com"
theme = "<testing-theme-id>"

[environments.uat]
store = "<store-name>.myshopify.com"
theme = "<uat-theme-id>"

[environments.production]
store = "<store-name>.myshopify.com"
theme = "<live-theme-id>"
```

Any CLI flag can be set in an environment block using its long name (e.g. `theme`, `ignore`, `nodelete`). Run `shopify theme list -e development` to find theme IDs.

> **Don't commit secrets.** If you use a [Theme Access](https://apps.shopify.com/theme-access) password, set it through the `SHOPIFY_CLI_THEME_TOKEN` environment variable instead of the `password` key in `shopify.theme.toml`.

Other useful commands:

| Command | Purpose |
| --- | --- |
| `shopify theme check` | Lint the theme for errors and best-practice issues |
| `shopify theme list -e development` | List themes on the store |
| `shopify theme pull -e production` | Pull theme files from a theme (e.g. settings changed in the editor) |
| `shopify theme push -e development --unpublished` | Push to a new unpublished theme for review |
| `shopify theme share -e development` | Upload to an unpublished theme and get a shareable preview link |

Because deploys to the connected testing, UAT, and live themes go through GitHub (see below), use `push` only for unpublished or personal preview themes.

## Branches & deployment

The store's themes are connected to GitHub branches through the Shopify GitHub integration (**Online Store → Themes → Add theme → Connect from GitHub**). Once connected, the branch and theme stay in sync in both directions:

- Commits pushed to the branch are deployed to the connected theme automatically.
- Changes made in the theme editor (customizer settings, JSON templates) are committed back to the branch by Shopify.

| Branch | Connected theme | Purpose |
| --- | --- | --- |
| `production` | Live (published) theme | **Source of truth.** What customers see. |
| `uat` | UAT theme (unpublished) | User acceptance testing: stakeholders review features before release. |
| `testing` | Testing theme (unpublished) | Developer testing on a real store theme. |
| `feature/<feature-name>` | None | One branch per feature, e.g. `feature/case-size-selector`. |

### Workflow

Every feature gets its own branch, which is promoted through each environment by pull request:

```
                        ┌──PR──▶  testing     (1. developer testing)
feature/<feature-name> ─┼──PR──▶  uat         (2. user testing)
                        └──PR──▶  production  (3. release)
```

1. **Branch off `production`** so you start from what's live:

   ```sh
   git checkout production && git pull
   git checkout -b feature/<feature-name>
   ```

2. **Develop locally** with `shopify theme dev -e development`, and run `shopify theme check` before opening a PR.
3. **Test:** open a PR from `feature/<feature-name>` into `testing`. Once merged, it deploys to the testing theme for developer testing.
4. **UAT:** once it passes testing, open a PR from `feature/<feature-name>` into `uat` for user testing.
5. **Release:** once it's approved in UAT, open a PR from `feature/<feature-name>` into `production` to go live.
6. Delete the feature branch after it's merged into `production`.

Merge the **feature branch** into each environment, not `testing` into `uat` or `uat` into `production`. That way, only features that have passed each stage get promoted, and unfinished work sitting in `testing` never reaches the live store.

Periodically reset `testing` and `uat` to match `production` so they don't drift:

```sh
git checkout testing && git reset --hard origin/production && git push --force-with-lease
```

> **Note:** Shopify commits theme editor changes back to the connected branch. Changes made in the live theme's editor land on `production`, so always `git pull` before branching, and expect conflicts in `templates/*.json` and `config/settings_data.json`. If you add GitHub branch protection to `production`, allow the Shopify GitHub app to bypass it, or editor changes can't sync back.

## Theme structure

```
assets/       CSS, JS, images, fonts
config/       Theme settings schema and saved settings data
layout/       Theme layouts (theme.liquid, password.liquid)
locales/      Translation strings
sections/     Reusable, customizable sections
snippets/     Reusable Liquid partials
templates/    JSON/Liquid page templates
shopify.theme.toml   Shopify CLI environments (store, theme IDs)
```
