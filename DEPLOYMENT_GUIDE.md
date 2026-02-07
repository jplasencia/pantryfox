# Deployment Guide: Pantry Fox Alexa Auth Page

## Quick Setup (5 steps, ~10 minutes)

### Step 1: Create GitHub Repository (2 minutes)

1. Go to: https://github.com/new
2. Create a new repository:
   - **Repository name**: `pantryfox-auth`
   - **Visibility**: Public (required for GitHub Pages)
   - **Do NOT** initialize with README, .gitignore, or license
3. Click "Create repository"

### Step 2: Push Code to GitHub (1 minute)

Run these commands in your terminal:

```bash
cd "/mnt/c/Users/Daniel Plasencia/sites/grocerylist_app/alexa-auth-page"
git push -u origin main
```

### Step 3: Enable GitHub Pages (2 minutes)

1. Go to: https://github.com/dplasencia/pantryfox-auth/settings/pages
2. Under "Source", select:
   - **Source**: Deploy from a branch
   - **Branch**: `main`
   - **Folder**: `/ (root)`
3. Click **Save**
4. Wait for the site to deploy (green checkmark appears)

### Step 4: Configure GoDaddy DNS (3 minutes)

1. Log in to: https://godaddy.com
2. Go to **My Products** → Click **DNS** next to `pantryfox.io`
3. **Delete** any existing A records for `@`
4. **Add** these 4 A records:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | 185.199.108.153 | 1 hour |
| A | @ | 185.199.109.153 | 1 hour |
| A | @ | 185.199.110.153 | 1 hour |
| A | @ | 185.199.111.153 | 1 hour |

5. **Add** a CNAME for www (optional):

| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | www | dplasencia.github.io | 1 hour |

6. Click **Save**

### Step 5: Configure Custom Domain in GitHub (1 minute)

1. Go to: https://github.com/dplasencia/pantryfox-auth/settings/pages
2. Under "Custom domain", enter: `pantryfox.io`
3. Click **Save**
4. Wait for DNS check to pass (may take a few minutes)
5. Click **...** → Enforce HTTPS (recommended)

---

## Features

The auth page supports two linking methods:

### 1. Google Sign-In (Recommended)
- Uses Supabase Auth's built-in Google OAuth
- No password required
- Works with existing Google accounts

### 2. Email Linking
- Enter email address directly
- Account must exist in Pantry Fox database
- Simpler for users without Google accounts

---

## Step 6: Configure Supabase Google Provider

Before using Google Sign-In, you need to enable Google OAuth in Supabase:

1. Go to: https://supabase.com/dashboard/project/nmomkpwelcposjhdurby/auth/providers
2. Enable **Google** provider
3. Add authorized redirect URL:
   - `https://pantryfox.io/`
4. Click **Save**

---

## Step 7: Update Alexa Developer Console

1. Go to: https://developer.amazon.com/alexa/console/ask
2. Open your **Pantry Fox** skill
3. Go to **Build** → **Account Linking**
4. Update settings:

| Field | Value |
|-------|-------|
| Authorization URI | `https://pantryfox.io/` |
| Access Token URI | `https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox-auth/token` |
| Client ID | `pantry-fox-alexa` |
| Client Secret | `pantry-fox-secret` |
| Your Authentication Scope | (leave blank) |
| Domain List | `pantryfox.io` |

5. **Allowed Return URLs** (should already be set):
   - `https://layla.amazon.com/api/skill/link/M30QOVU46N9ND`
   - `https://pitangui.amazon.com/api/skill/link/M30QOVU46N9ND`
   - `https://alexa.amazon.co.jp/api/skill/link/M30QOVU46N9ND`

6. Click **Save**

---

## Testing

Once DNS has propagated (usually 5-30 minutes):

1. Test the auth page:
   ```
   https://pantryfox.io/?client_id=pantry-fox-alexa&response_type=code&redirect_uri=https://layla.amazon.com/api/skill/link/M30QOVU46N9ND&state=test123
   ```

2. Test Google Sign-In:
   - Click "Sign in with Google"
   - Complete Google OAuth flow
   - Should redirect back to Alexa successfully

3. Test Email option:
   - Enter your email address
   - Click "Link Account"
   - Should redirect back to Alexa successfully

4. In the Alexa app:
   - Go to Your Skills → Pantry Fox
   - Click "Link Account"
   - Choose Google Sign-In or enter email
   - Should redirect back to Alexa successfully

---

## Troubleshooting

### DNS not working
- Run: `dig pantryfox.io` (Mac/Linux) or `nslookup pantryfox.io` (Windows)
- Should return the GitHub Pages IPs (185.199.x.x)
- DNS can take 1-48 hours to propagate, but usually within 30 minutes

### GitHub Pages shows "Page not found"
- Make sure repository is Public
- Wait a few minutes after pushing
- Check Settings → Pages for deployment status

### Alexa shows "There was a problem"
- Check Alexa Developer Console for errors
- Verify all URLs are correct
- Check Supabase logs for Edge Function errors

---

## Files Ready

✓ `index.html` - Styled auth page
✓ `CNAME` - Custom domain configuration
✓ Git repository initialized
✓ Remote configured to: `https://github.com/dplasencia/pantryfox-auth.git`

---

## Command Reference

```bash
# Push to GitHub (run after creating repo)
cd "/mnt/c/Users/Daniel Plasencia/sites/grocerylist_app/alexa-auth-page"
git push -u origin main

# Check deployment status
# Visit: https://github.com/dplasencia/pantryfox-auth/settings/pages

# Check DNS propagation
# Visit: https://dnschecker.org/#A/pantryfox.io
```
