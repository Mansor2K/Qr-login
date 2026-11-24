# QR code login: backend changes

Copy these files into `back-jasmen-water/` (same paths), then:

    npm install            # adds qrcode + express-rate-limit
    # add to .env:
    QR_LOGIN_BASE_URL=https://jasminewaterflow.com/qr-login

If the server runs behind nginx or another proxy, add `app.set('trust proxy', 1);`
in index.js right after `const app = express();` so rate limiting and the
stored IP use the real client address.

## Endpoints (all under /auth/v1)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | /qr/issue | admin (access_token header) | body `{ userId, expiresInMinutes? }`; returns `{ id, expiresAt, link, qrDataUrl, user }` |
| POST | /qr/redeem | public, 20 per 15 min per IP | body `{ token }`; same response as /Login (access_token header + `{role, user}`) |
| GET | /qr/user/:userId | admin | list codes issued for a user (no secrets) |
| DELETE | /qr/:id | admin | revoke an unused code |

## Frontend contract

The QR encodes `QR_LOGIN_BASE_URL#t=<token>`. The page at that URL reads the
token from `location.hash`, immediately POSTs it to /auth/v1/qr/redeem, stores the
`access_token` response header exactly as the normal login does, then clears the hash.
