# Dubai Mall — Bruno Collection

Every endpoint the web app talks to, as a native [Bruno](https://usebruno.com) collection, with **Staging** and **Production** environments.

## Open it

Bruno → **Open Collection** → select this `bruno/` folder. Then pick **Staging** or **Production** from the environment dropdown (top-right) before sending.

## What's inside (37 requests)

| Folder | Notes |
| --- | --- |
| Malls, Whats New - Ads, Events, Stores and Dining, Taxonomy, Parking and Valet, Pages | Plain `GET` against `{{API_URL}}` with the `X-Token` header. v8 requests use `{{API_TOKEN_V8}}`. |
| V3 and Partners (JWT POST) | `POST` with a signed JWT body — signed automatically, see below. |
| Rewards - U-by-Emaar (upstream) | UBE + OneApp, authenticated with `x-api-key`. |
| App Routes (Next.js) | The app's own routes (`/api/revalidate`, `/api/emaar/*`, catch-all proxy). Point `{{APP_URL}}` at the running app. |

## JWT endpoints (v3/\* and v1/partners)

These POST a signed HS256 JWT as the request body (`Content-Type: text/plain`), mirroring `signJwt()` in `lib/api.ts`. The folder's **pre-request script** signs `{ lang, token_type }` with `{{JWT_SECRET}}` (using the built-in `crypto-js`) and stores it in the `jwt` runtime var that each request sends as `{{jwt}}` — nothing to do manually. It's verified to produce a byte-identical token to the server's signer.

The **response** is also JWT-encoded: base64url-decode the middle segment to read the `data` payload (as `decodeJwtPayload` does in the app).

> If you see `JWT_SECRET is not set`, you haven't selected an environment — pick Staging or Production first.

## Environment differences

| Variable | Staging | Production |
| --- | --- | --- |
| `API_URL` | `https://apidev.emaar.com/mcpstg` | `https://mypage.emaarmalls.com` |
| `UBE_API_URL` | `https://apidev.emaar.com/v1/ube-website` | `https://api.emaar.com/v1/ube-website` |
| `UBE_API_KEY` | staging key | prod key |
| `APP_URL` | `http://localhost:3000` | `https://thedubaimall.com` |

`API_TOKEN`, `API_TOKEN_V8`, `JWT_SECRET`, `TOKEN_TYPE`, `ONEAPP_*` are the same in both (pulled from `.env.staging` / `.env.prod`).

> `API_TOKEN_V8` in production is still the **staging** token — `.env.prod` has a `TODO` to replace it with the real prod v8 token. Update it in the Production environment once you have it. It affects the v8 requests (top/entertain stores, valet).

Per-request variables: `{{lang}}` (`en`/`ar`), `{{ube_lang}}` (`en-ae`), `{{slug}}` (for *Get Page Partner*).

## ⚠️ Secrets

The environment `.bru` files contain live API tokens, the JWT secret, and API keys (taken straight from the repo's `.env` files). Don't commit this `bruno/` folder to a public repo or share the environment files outside the team.
