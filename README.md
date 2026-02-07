# Pantry Fox Alexa Authentication Page

This page handles OAuth 2.0 authorization for Alexa account linking with Pantry Fox.

## Features

- **Google Sign-In**: Quick authentication using Supabase Auth + Google OAuth
- **Email Linking**: Alternative method using email address directly
- **Beautiful UI**: Modern gradient design with smooth animations

## Deployment

### 1. Create GitHub Repository

1. Create a new GitHub repository (e.g., `pantryfox-auth` or `pantryfox-io`)
2. Upload the `index.html` file to the repository
3. Enable GitHub Pages in repository Settings:
   - Go to Settings → Pages
   - Source: Deploy from a branch
   - Branch: `main` or `master`, folder: `/ (root)`
   - Save

### 2. Configure GoDaddy DNS

1. Log in to your GoDaddy account at https://godaddy.com
2. Go to **My Products** → **DNS** next to pantryfox.io
3. Add the following DNS records:

#### For root domain (pantryfox.io):
| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | 185.199.108.153 | 1 hour |
| A | @ | 185.199.109.153 | 1 hour |
| A | @ | 185.199.110.153 | 1 hour |
| A | @ | 185.199.111.153 | 1 hour |

#### For www subdomain:
| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | www | pantryfox.github.io | 1 hour |

*Note: The IP addresses above are GitHub Pages' official IPs.*

4. Wait for DNS propagation (can take 1-48 hours, usually within minutes)

### 3. Configure Custom Domain in GitHub

1. Go to your repository's **Settings** → **Pages**
2. Under "Custom domain", enter: `pantryfox.io`
3. Click **Save**
4. Wait for DNS check to pass
5. Enforce HTTPS (recommended)

### 4. Update Alexa Developer Console

Once the auth page is live at `https://pantryfox.io`, update the Alexa skill configuration:

1. Go to Alexa Developer Console: https://developer.amazon.com/alexa/console/ask
2. Open your "Pantry Fox" skill
3. Go to **Build** → **Account Linking**
4. Update the following:

**Authorization URI:**
```
https://pantryfox.io/?client_id=pantry-fox-alexa&response_type=code&redirect_uri={redirect_uri}&state={state}
```

**Access Token URI:**
```
https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox-auth/token
```

**Client ID:** `pantry-fox-alexa`
**Client Secret:** `pantry-fox-secret`

**Allowed Return URLs:**
- `https://layla.amazon.com/api/skill/link/M30QOVU46N9ND`
- `https://pitangui.amazon.com/api/skill/link/M30QOVU46N9ND`
- `https://alexa.amazon.co.jp/api/skill/link/M30QOVU46N9ND`

5. Save changes

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Alexa                                    │
└─────────────────────────────────────────────────────────────────┘
                            │
                            │ 1. User clicks "Link Account"
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│              GitHub Pages (pantryfox.io)                         │
│              - Shows login form                                  │
│              - Collects email                                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            │ 2. POST /api/link-account with email
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│         Supabase Edge Function (pantry-fox-auth)                │
│         - Validates email in database                           │
│         - Generates auth code                                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            │ 3. Redirect with code
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Alexa                                    │
│         - Exchanges code for access token                       │
└─────────────────────────────────────────────────────────────────┘
                            │
                            │ 4. POST /token
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│         Supabase Edge Function (pantry-fox-auth)                │
│         - Returns access token                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Testing

1. Visit `https://pantryfox.io/?client_id=pantry-fox-alexa&response_type=code&redirect_uri=https://layla.amazon.com/api/skill/link/M30QOVU46N9ND&state=test123`
2. Enter a valid email from your Pantry Fox users table
3. Submit form - should redirect to Alexa with auth code

## Troubleshooting

**DNS not propagating:**
- Use `dig pantryfox.io` or `nslookup pantryfox.io` to check
- Can take up to 48 hours, but usually faster

**GitHub Pages DNS check failing:**
- Make sure all 4 A records are added
- Wait a bit longer for DNS propagation

**Auth page shows error:**
- Check browser console for errors
- Verify Supabase Edge Function is deployed and accessible
- Check that API_BASE_URL in index.html is correct
# pantryfox
