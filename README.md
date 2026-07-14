[![Deploy on Bot-Hosting](https://bot-hosting.net/assets/deploy-badge.svg)](https://bot-hosting.net/deploy?source=template&template=discord-login-flow)

# Discord Login Template

An Express template that adds a "Login with Discord" flow (OAuth2) and a protected dashboard page, using plain HTML/CSS/JS on the frontend — no templating engine required.

## How it works

1. User clicks **Login with Discord** on `index.html`, which sends them to `/auth/discord`.
2. That route redirects them to Discord's consent screen.
3. After approving, Discord redirects back to `/auth/discord/callback` with a code.
4. The server exchanges that code for an access token, fetches the user's Discord profile, and stores it in a server-side session.
5. The user is redirected to `/dashboard`, which calls `/api/user` to fetch and display their info.
6. `/auth/logout` destroys the session and sends them back to the login page.

`/dashboard` is protected — if there's no active session, requesting it redirects straight back to `/`. Static files are served without their `.html` extension (`express.static` is configured with `extensions: ['html']`), so `/dashboard` maps to `public/dashboard.html` and `/` maps to `public/index.html`.

## Getting Started

### Create a Discord application

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) and click **New Application**.
2. Under **OAuth2 → General**, copy your **Client ID** and **Client Secret**.
3. Under **OAuth2 → Redirects**, add:
   ```
   http://localhost:3000/auth/discord/callback
   ```
   (update the port/domain if you change it later)

### 3. Set up your environment variables

Copy the example env file:

```bash
cp example.env .env
```

Then fill in `.env`:

```
PORT=3000
DISCORD_CLIENT_ID=your_client_id
DISCORD_CLIENT_SECRET=your_client_secret
DISCORD_REDIRECT_URI=http://localhost:3000/auth/discord/callback
SESSION_SECRET=any_long_random_string
```

## Project Structure

```
.
├── index.js                  # Express server, session setup, OAuth routes
├── example.env               # Rename to .env in production
├── package.json
├── example.env
└── public/
    ├── index.html             # Login page
    ├── dashboard.html         # Protected dashboard (client-side rendered)
    └── css/
        └── styles.css
```

## Customizing

- **Scopes** — the login route requests `identify email`. Add more scopes (e.g. `guilds`) in the `scope` parameter inside the `/auth/discord` route in `index.js`, and update the callback to fetch/store any extra data you need.
- **Session storage** — sessions currently live in memory, which resets on server restart and won't scale across multiple server instances. For production, swap in a persistent session store (e.g. `connect-redis` or `connect-mongo`).
- **Dashboard content** — `dashboard.html` currently shows username, tag, user ID, and email. Extend the `/api/user` route in `index.js` to return more data, and update the script in `dashboard.html` to display it.
- **Styling** — all styles are in `public/css/styles.css`.

## Notes

- Requires Node 18+ (uses the built-in global `fetch`).
- Never commit your real `.env` file — only `example.env` should be checked into version control.
