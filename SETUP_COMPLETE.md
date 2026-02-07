# Alexa Account Linking - Complete Setup Summary

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Alexa Account Linking Flow                        │
└─────────────────────────────────────────────────────────────────────────────┘

1. User clicks "Link Account" in Alexa App
                    │
                    ▼
2. Alexa redirects to: https://pantryfox.io/
   (with client_id, redirect_uri, state, response_type=code)
                    │
                    ▼
3. User chooses sign-in method:
   ┌─────────────────────────────────────────┐
   │  a) Google Sign-In                      │
   │     → Supabase Auth + Google OAuth      │
   │     → Returns user email                │
   │                                         │
   │  b) Email Linking                       │
   │     → User enters email directly        │
   └─────────────────────────────────────────┘
                    │
                    ▼
4. Auth page calls: /api/link-account
   - Validates email exists in database
   - Generates auth code
   - Stores: authCode → userId mapping
                    │
                    ▼
5. Auth page redirects to Alexa with auth code
                    │
                    ▼
6. Alexa exchanges code for access token via: /token
   - Returns access token (base64 encoded userId)
                    │
                    ▼
7. Future skill requests include access token
   - Skill decodes token → userId
   - Creates alexa_users mapping
   - Serves personalized responses
```

## Deployed Components

### 1. GitHub Pages Auth Page
- **Repo**: `https://github.com/dplasencia/pantryfox-auth`
- **Domain**: `https://pantryfox.io`
- **Features**:
  - Google Sign-In (Supabase Auth)
  - Email linking
  - Beautiful gradient UI

### 2. Supabase Edge Functions

#### pantry-fox-auth (OAuth Handler)
- **URL**: `https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox-auth`
- **Endpoints**:
  - `/auth` - Login page (now served by GitHub Pages instead)
  - `/api/link-account` - Store auth code → userId mapping
  - `/token` - Exchange auth code for access token

#### pantry-fox (Main Skill)
- **URL**: `https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox`
- **Features**:
  - Handles Alexa skill requests
  - Decodes access tokens
  - Manages alexa_users mapping

## Configuration Checklist

### ✅ Completed
- [x] Auth page created with Google Sign-In
- [x] Auth page created with email option
- [x] Auth Edge Function deployed (`pantry-fox-auth`)
- [x] Main skill Edge Function deployed (`pantry-fox`)
- [x] GitHub repo initialized
- [x] CNAME configured for pantryfox.io
- [x] Deployment guide created

### ⏳ Pending (User Action Required)

#### 1. Push to GitHub
```bash
cd "/mnt/c/Users/Daniel Plasencia/sites/grocerylist_app/alexa-auth-page"
git push -u origin main
```
Use GitHub Personal Access Token for password.

#### 2. Enable GitHub Pages
1. Go to: https://github.com/dplasencia/pantryfox-auth/settings/pages
2. Source: Branch `main`, folder `/ (root)`
3. Save
4. Wait for deployment

#### 3. Configure Supabase Google Provider
1. Go to: https://supabase.com/dashboard/project/nmomkpwelcposjhdurby/auth/providers
2. Enable **Google** provider
3. Add redirect URL: `https://pantryfox.io/`
4. Save

#### 4. Verify DNS (Already Done per User)
- GoDaddy A records configured for pantryfox.io
- Custom domain added in GitHub Pages settings

#### 5. Update Alexa Developer Console
1. Go to: https://developer.amazon.com/alexa/console/ask
2. Open **Pantry Fox** skill
3. Build → Account Linking
4. Authorization URI: `https://pantryfox.io/`
5. Access Token URI: `https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox-auth/token`
6. Client ID: `pantry-fox-alexa`
7. Client Secret: `pantry-fox-secret`
8. Domain List: `pantryfox.io`
9. Save

## Test URLs

### Test Auth Page
```
https://pantryfox.io/?client_id=pantry-fox-alexa&response_type=code&redirect_uri=https://layla.amazon.com/api/skill/link/M30QOVU46N9ND&state=test123
```

### Test Direct Link-Account API
```bash
curl -X POST https://nmomkpwelcposjhdurby.supabase.co/functions/v1/pantry-fox-auth/api/link-account \
  -H "Content-Type: application/json" \
  -d '{"email":"your@email.com","authCode":"test123"}'
```

## Files Structure

```
alexa-auth-page/
├── index.html          # Main auth page (Google + Email)
├── CNAME              # Custom domain for GitHub Pages
├── README.md          # Project documentation
├── DEPLOYMENT_GUIDE.md # Step-by-step setup guide
└── SETUP_COMPLETE.md  # This file

supabase/edge_functions/
├── pantry-fox-auth/
│   ├── index.ts       # OAuth handler (link-account, token)
│   └── deno.json
└── pantry-fox/
    ├── index.ts       # Main Alexa skill
    └── deno.json
```

## Security Notes

1. **Access Token Format**: `base64(userId_timestamp)`
2. **Auth Code Expiry**: 10 minutes
3. **Access Token Expiry**: 1 year
4. **OAuth Redirects**: Only Alexa domains allowed
5. **Client Secret**: Shared secret between auth page and Alexa

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Auth page not loading | Check GitHub Pages deployment status |
| Google Sign-In fails | Enable Google provider in Supabase Auth settings |
| "Account not found" | User must exist in `users` table first |
| Alexa link fails | Check Account Linking configuration in Developer Console |
| Access token invalid | Check `/token` endpoint is working |

## Next Steps After Setup

1. **Test the full flow**:
   - Open Alexa app
   - Go to Your Skills → Pantry Fox
   - Click "Link Account"
   - Try both Google Sign-In and Email options

2. **Monitor logs**:
   - Supabase Dashboard → Logs
   - Check for errors in Edge Functions

3. **Update users table**:
   - Ensure test users exist in the database
   - Use valid emails from your `users` table
