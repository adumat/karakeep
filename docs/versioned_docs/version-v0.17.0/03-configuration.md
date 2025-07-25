# Configuration

The app is mainly configured by environment variables. All the used environment variables are listed in [packages/shared/config.ts](https://github.com/hoarder-app/hoarder/blob/main/packages/shared/config.ts). The most important ones are:

| Name                      | Required                              | Default | Description                                                                                                                                    |
| ------------------------- | ------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| DATA_DIR                  | Yes                                   | Not set | The path for the persistent data directory. This is where the db and the uploaded assets live.                                                 |
| NEXTAUTH_URL              | Yes                                   | Not set | Should point to the address of your server. The app will function without it, but will redirect you to wrong addresses on signout for example. |
| NEXTAUTH_SECRET           | Yes                                   | Not set | Random string used to sign the JWT tokens. Generate one with `openssl rand -base64 36`.                                                        |
| MEILI_ADDR                | No                                    | Not set | The address of meilisearch. If not set, Search will be disabled. E.g. (`http://meilisearch:7700`)                                              |
| MEILI_MASTER_KEY          | Only in Prod and if search is enabled | Not set | The master key configured for meilisearch. Not needed in development environment. Generate one with `openssl rand -base64 36 \| tr -dc 'A-Za-z0-9'`                   |
| MAX_ASSET_SIZE_MB         | No                                    | 4       | Sets the maximum allowed asset size (in MB) to be uploaded                                                                                     |
| DISABLE_NEW_RELEASE_CHECK | No                                    | false   | If set to true, latest release check will be disabled in the admin panel.                                                                      |

## Authentication / Signup

By default, Hoarder uses the database to store users, but it is possible to also use OAuth.
The flags need to be provided to the `web` container.

:::info
Only OIDC compliant OAuth providers are supported! For information on how to set it up, consult the documentation of your provider.
:::

:::info
When setting up OAuth, the allowed redirect URLs configured at the provider should be set to `<HOARDER_ADDRESS>/api/auth/callback/custom` where `<HOARDER_ADDRESS>` is the address you configured in `NEXTAUTH_URL` (for example: `https://try.hoarder.app/api/auth/callback/custom`).
:::

| Name                                        | Required | Default                | Description                                                                                                                                                         |
| ------------------------------------------- | -------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DISABLE_SIGNUPS                             | No       | false                  | If enabled, no new signups will be allowed and the signup button will be disabled in the UI                                                                         |
| OAUTH_WELLKNOWN_URL                         | No       | Not set                | The "wellknown Url" for openid-configuration as provided by the OAuth provider                                                                                      |
| OAUTH_CLIENT_SECRET                         | No       | Not set                | The "Client Secret" as provided by the OAuth provider                                                                                                               |
| OAUTH_CLIENT_ID                             | No       | Not set                | The "Client ID" as provided by the OAuth provider                                                                                                                   |
| OAUTH_SCOPE                                 | No       | "openid email profile" | "Full list of scopes to request (space delimited)"                                                                                                                  |
| OAUTH_PROVIDER_NAME                         | No       | "Custom Provider"      | The name of your provider. Will be shown on the signup page as "Sign in with `<name>`"                                                                              |
| OAUTH_ALLOW_DANGEROUS_EMAIL_ACCOUNT_LINKING | No       | false                  | Whether existing accounts in hoarder stored in the database should automatically be linked with your OAuth account. Only enable it if you trust the OAuth provider! |

For more information on `OAUTH_ALLOW_DANGEROUS_EMAIL_ACCOUNT_LINKING`, check the [next-auth.js documentation](https://next-auth.js.org/configuration/providers/oauth#allowdangerousemailaccountlinking-option).

## Inference Configs (For automatic tagging)

Either `OPENAI_API_KEY` or `OLLAMA_BASE_URL` need to be set for automatic tagging to be enabled. Otherwise, automatic tagging will be skipped.

:::warning

- The quality of the tags you'll get will depend on the quality of the model you choose.
- Running local models is a recent addition and not as battle tested as using OpenAI, so proceed with care (and potentially expect a bunch of inference failures).
  :::

| Name                      | Required | Default     | Description                                                                                                                                                                                     |
| ------------------------- | -------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OPENAI_API_KEY            | No       | Not set     | The OpenAI key used for automatic tagging. More on that in [here](/openai).                                                                                                                     |
| OPENAI_BASE_URL           | No       | Not set     | If you just want to use OpenAI you don't need to pass this variable. If, however, you want to use some other openai compatible API (e.g. azure openai service), set this to the url of the API. |
| OLLAMA_BASE_URL           | No       | Not set     | If you want to use ollama for local inference, set the address of ollama API here.                                                                                                              |
| OLLAMA_KEEP_ALIVE         | No       | Not set     | Controls how long the model will stay loaded into memory following the request (example value: "5m").                                                                                           |
| INFERENCE_TEXT_MODEL      | No       | gpt-4o-mini | The model to use for text inference. You'll need to change this to some other model if you're using ollama.                                                                                     |
| INFERENCE_IMAGE_MODEL     | No       | gpt-4o-mini | The model to use for image inference. You'll need to change this to some other model if you're using ollama and that model needs to support vision APIs (e.g. llava).                           |
| INFERENCE_LANG            | No       | english     | The language in which the tags will be generated.                                                                                                                                               |
| INFERENCE_JOB_TIMEOUT_SEC | No       | 30          | How long to wait for the inference job to finish before timing out. If you're running ollama without powerful GPUs, you might want to increase the timeout a bit.                               |

## Crawler Configs

| Name                          | Required | Default | Description                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------------------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CRAWLER_NUM_WORKERS           | No       | 1       | Number of allowed concurrent crawling jobs. By default, we're only doing one crawling request at a time to avoid consuming a lot of resources.                                                                                                                                                                                                                                     |
| BROWSER_WEB_URL               | No       | Not set | The browser's http debugging address. The worker will talk to this endpoint to resolve the debugging console's websocket address. If you already have the websocket address, use `BROWSER_WEBSOCKET_URL` instead. If neither `BROWSER_WEB_URL` nor `BROWSER_WEBSOCKET_URL` are set, the worker will launch its own browser instance (assuming it has access to the chrome binary). |
| BROWSER_WEBSOCKET_URL         | No       | Not set | The websocket address of browser's debugging console. If you want to use [browserless](https://browserless.io), use their websocket address here. If neither `BROWSER_WEB_URL` nor `BROWSER_WEBSOCKET_URL` are set, the worker will launch its own browser instance (assuming it has access to the chrome binary).                                                                 |
| BROWSER_CONNECT_ONDEMAND      | No       | false   | If set to false, the crawler will proactively connect to the browser instance and always maintain an active connection. If set to true, the browser will be launched on demand only whenever a crawling is requested. Set to true if you're using a service that provides you with browser instances on demand.                                                                    |
| CRAWLER_DOWNLOAD_BANNER_IMAGE | No       | true    | Whether to cache the banner image used in the cards locally or fetch it each time directly from the website. Caching it consumes more storage space, but is more resilient against link rot and rate limits from websites.                                                                                                                                                         |
| CRAWLER_STORE_SCREENSHOT      | No       | true    | Whether to store a screenshot from the crawled website or not. Screenshots act as a fallback for when we fail to extract an image from a website. You can also view the stored screenshots for any link.                                                                                                                                                                           |
| CRAWLER_FULL_PAGE_SCREENSHOT  | No       | false   | Whether to store a screenshot of the full page or not. Disabled by default, as it can lead to much higher disk usage. If disabled, the screenshot will only include the visible part of the page                                                                                                                                                                                   |
| CRAWLER_FULL_PAGE_ARCHIVE     | No       | false   | Whether to store a full local copy of the page or not. Disabled by default, as it can lead to much higher disk usage. If disabled, only the readable text of the page is archived.                                                                                                                                                                                                 |
| CRAWLER_JOB_TIMEOUT_SEC       | No       | 60      | How long to wait for the crawler job to finish before timing out. If you have a slow internet connection or a low powered device, you might want to bump this up a bit                                                                                                                                                                                                             |
| CRAWLER_NAVIGATE_TIMEOUT_SEC  | No       | 30      | How long to spend navigating to the page (along with its redirects). Increase this if you have a slow internet connection                                                                                                                                                                                                                                                          |
