# AM. Store - Security Deployment Guide

## Overview

This guide explains how to deploy the security improvements for your AM. store.

---

## What Was Fixed

### 1. Removed Hardcoded Credentials
- **Before:** Admin passwords (`lmaoyounoob`, `hamza`, `AM-BACKUP-0000`) were visible to anyone
- **After:** All credentials are now validated server-side through Supabase

### 2. Improved Password Hashing
- **Before:** Weak custom hash function (easily reversible)
- **After:** SHA-256 cryptographic hashing via Web Crypto API

### 3. Server-Side Authentication
- **Before:** Admin login checked credentials in the browser (insecure)
- **After:** Login queries the database server-side (secure)

### 4. Email API Key Protected
- **Before:** Brevo API key was exposed in client-side code
- **After:** API key is only accessed through Supabase database

---

## Deployment Steps

### Step 1: Update Your index.html

Replace your current `index.html` with the updated version from:
```
C:\Users\hater\Downloads\index.html
```

### Step 2: Run RLS Policies in Supabase

1. Go to [Supabase Dashboard](https://supabase.com/dashboard)
2. Select your project (`afwsaqsagrlzxqjaaovc`)
3. Go to **SQL Editor**
4. Copy and paste the contents of:
   ```
   C:\Users\hater\Downloads\supabase\migrations\001_enable_rls.sql
   ```
5. Click **Run**

### Step 3: Deploy Edge Functions (Optional but Recommended)

If you have Supabase CLI installed:

```bash
# Install Supabase CLI
npm install -g supabase

# Login to Supabase
supabase login

# Link to your project
supabase link --project-ref afwsaqsagrlzxqjaaovc

# Deploy functions
supabase functions deploy admin-auth
supabase functions deploy place-order
supabase functions deploy send-email
```

### Step 4: Set Environment Variables in Supabase

1. Go to **Project Settings** → **Edge Functions**
2. Add these environment variables:
   - `SUPABASE_URL`: `https://afwsaqsagrlzxqjaaovc.supabase.co`
   - `SUPABASE_ANON_KEY`: Your anon key (same as in index.html)

### Step 5: Update Your Password Hash

**Important:** After deploying, you need to update your admin password hash.

1. Open browser console on your site
2. Run this to generate a new hash:
   ```javascript
   hashPassword('your-new-password').then(h => console.log(h));
   ```
3. Copy the output
4. Go to your Supabase Dashboard → **Table Editor** → **store_config**
5. Find the row with `key = 'settings'`
6. Update the `password` field with the new hash

### Step 6: Push to GitHub

```bash
cd C:\Users\hater\Downloads
git add .
git commit -m "Security: Remove hardcoded credentials, add SHA-256 hashing"
git push origin main
```

---

## GitHub Branch Strategy (Recommended)

To control when changes go live:

### Option A: Two-Branch Workflow

```bash
# Create development branch
git checkout -b development

# Make changes on development branch
# ... edit files ...

# When ready to deploy:
git checkout main
git merge development
git push origin main
```

### Option B: GitHub Actions Auto-Deploy

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: .
```

---

## Testing Your Security

### Test 1: Check No Credentials in Source
1. Right-click your website → **View Page Source**
2. Search for "lmaoyounoob" or "hamza"
3. **Result:** Should NOT find these strings

### Test 2: Test Admin Login
1. Go to your admin panel
2. Enter your username and password
3. **Result:** Login should work normally

### Test 3: Check Console for Errors
1. Open browser DevTools (F12)
2. Go to **Console** tab
3. Try logging in
4. **Result:** No security errors

---

## Files Created

| File | Purpose |
|------|---------|
| `supabase/functions/admin-auth/index.ts` | Server-side admin authentication |
| `supabase/functions/place-order/index.ts` | Secure order placement |
| `supabase/functions/send-email/index.ts` | Email sending (protects API key) |
| `supabase/migrations/001_enable_rls.sql` | Database security policies |
| `_headers` | Security headers for hosting |
| `SECURITY_DEPLOY.md` | This guide |

---

## What's Still Exposed (Acceptable)

The Supabase URL and anon key are still in your client-side code. This is **normal and acceptable** because:

1. **RLS protects your database** - Even with the anon key, unauthorized access is blocked
2. **Edge Functions handle secrets** - API keys are only used server-side
3. **Admin panel is password-protected** - Cannot be bypassed from client side

---

## Need Help?

If you encounter issues:

1. Check the browser console for errors
2. Verify RLS policies are enabled in Supabase
3. Test the Edge Functions in Supabase Dashboard → Edge Functions → Logs
4. Make sure your admin password hash is updated

---

## Security Checklist

- [ ] RLS policies enabled in Supabase
- [ ] Updated index.html deployed
- [ ] Admin password hash updated
- [ ] Old backup files deleted from server
- [ ] GitHub Pages only serves from main branch
- [ ] No sensitive files in public folder
