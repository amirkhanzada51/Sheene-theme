# update-theme.md

Sheene store, Horizon theme ko update karne ka process.

Store: `qx0nga-dg.myshopify.com`
Theme: Horizon (Shopify first party)
Upstream: https://github.com/Shopify/horizon

---

## Kyun ye process, admin ka Update button kyun nahi

Shopify admin ka "Update available" button ek **naya draft theme** banata hai. Wo draft tumhari GitHub branch se connected nahi hota, aur tumhare code edits reliably carry nahi hote. GitHub connected setup me us button ko **ignore karo**. Update hamesha git se aayega.

---

## Ek dafa ka setup

```bash
cd sheene-theme
git remote add upstream https://github.com/Shopify/horizon.git
git fetch upstream
git remote -v          # verify: origin = tumhara repo, upstream = Shopify/horizon
```

### Base commit pin karo

Horizon repo pe **koi release tags nahi hain**. Version number `release-notes.md` file me likha hota hai. To base ko commit SHA se pin karna hai.

1. Admin > Online Store > Themes. Theme ke naam ke neeche version number dekho, e.g. `4.1.3`.
2. Repo me us file ki history dekho:

```bash
git log --oneline upstream/main -- release-notes.md
```

3. Jis commit pe tumhara wala version tha, uska SHA lo aur tag lagao:

```bash
git tag horizon-base <SHA>
git push origin horizon-base
```

Exact version na mile to sabse qareeb commit le lo. Bas thoda extra diff aayega, kaam kharab nahi hoga.

---

## Update ka process

### 1. Clean start

```bash
git checkout dev
git pull                 # ZAROORI: theme editor ke auto commits utaar lo
git status               # working tree clean hona chahiye
git fetch upstream
```

### 2. Dekho kya aa raha hai

```bash
git log --oneline horizon-base..upstream/main
git diff --stat horizon-base upstream/main
```

Isse pata chalega kitni files change hui aur kitna bada update hai.

### 3. Safety branch

```bash
git checkout -b theme-update-$(date +%Y%m%d)
```

Direct `dev` pe update mat lagao. Alag branch pe lagao, test karo, phir merge.

### 4. Patch banao aur apply karo

```bash
git diff horizon-base upstream/main \
  -- . ':!config/settings_data.json' ':!config/settings_schema.json' > update.patch

git apply --3way update.patch
```

Do files deliberately exclude ki hain:

- `config/settings_data.json`: tumhari homepage sections, colors, saari theme editor settings isme hain. Vanilla version se replace hui to sab ud jayega.
- `config/settings_schema.json`: agar tumne custom settings add ki hain to wo yahan hain.

Agar Horizon ne genuinely naye settings add kiye hain, to `settings_schema.json` ko baad me **manually** dekho aur sirf naye blocks copy karo.

### 5. Conflicts resolve karo

```bash
git status                # conflicted files
git diff --name-only --diff-filter=U
```

Conflict sirf wahan aayega jahan tumne khud core file edit ki thi. Har file kholo, `<<<<<<<` markers dhundo, decide karo kya rakhna hai. Rule of thumb: **naya Horizon code base rakho, apna custom logic uske upar dobara lagao**, blind "ours" mat maro warna bug fixes miss ho jayenge.

### 6. Test

```bash
shopify theme check
shopify theme dev --store qx0nga-dg.myshopify.com
```

Manually check karo:

- Homepage, sections ka order aur content
- Product page, variants, add to cart
- Cart aur checkout tak ka flow
- Mobile view
- Jo bhi custom sections ya snippets tumne banaye the
- Browser console me koi JS error to nahi

### 7. Staging pe bhejo

```bash
git add -A
git commit -m "Update Horizon to <version>"
git checkout dev
git merge theme-update-<date>
git push origin dev
```

Staging theme ka preview link kholo, wapas se poora flow check karo. Yahan client ya team ko dikha do.

### 8. Live karo

```bash
git checkout main
git pull
git merge dev
git push origin main
```

Live update ho gaya. Publish button dabane ki zaroorat nahi.

### 9. Base aage badhao

```bash
git tag -f horizon-base upstream/main
git push -f origin horizon-base
```

Ye step bhoolna mat, warna agli dafa purana diff dobara apply hoga.

---

## Rollback

Agar live pe kuch toot jaye:

```bash
git checkout main
git revert -m 1 HEAD
git push origin main
```

Shopify khud rollback kar dega, 30 second me.

Emergency me admin > Themes > jo backup duplicate rakha tha usse Publish kar do. Isliye har update se pehle live theme ka **Duplicate** banana chahiye.

---

## Conflicts kam karne ka tareeqa

Horizon theme blocks pe bana hai. Jitna kam core files chhedoge, utna smooth update rahega.

- Custom kaam **naye files** me karo: naya section, naya block, naya snippet. Core file edit karne se bacho.
- CSS `assets/` me apni alag file me rakho, Horizon ki base CSS me mat likho.
- Core file edit karni hi pade to upar comment daal do, e.g. `{% comment %} CUSTOM: Sheene hero override {% endcomment %}`. Merge ke waqt turant nazar aayega.
- Jo bhi core file tumne edit ki hai, uski list `CUSTOMIZATIONS.md` me maintain karo. Update ke waqt sabse pehle wahi file kholo.

---

## Quick reference

| Kaam | Command |
|---|---|
| Upstream latest lao | `git fetch upstream` |
| Kya change hua dekho | `git diff --stat horizon-base upstream/main` |
| Patch banao | `git diff horizon-base upstream/main -- . ':!config/settings_data.json' ':!config/settings_schema.json' > update.patch` |
| Apply karo | `git apply --3way update.patch` |
| Conflicted files | `git diff --name-only --diff-filter=U` |
| Local test | `shopify theme dev --store qx0nga-dg.myshopify.com` |
| Lint | `shopify theme check` |
| Staging deploy | `git push origin dev` |
| Live deploy | `git checkout main && git merge dev && git push origin main` |
| Base update | `git tag -f horizon-base upstream/main && git push -f origin horizon-base` |
| Rollback | `git revert -m 1 HEAD && git push origin main` |
















# OLD WAY
git checkout vendor
# saari files replace karo naye vanilla version se
git add -A && git commit -m "vendor: Dawn 16.0.0"
git tag vendor-16.0.0

git checkout dev
git diff vendor-15.3.0 vendor-16.0.0 -- . ':!config/settings_data.json' > update.patch
git apply --3way update.patch
