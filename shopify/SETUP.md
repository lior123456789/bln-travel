# BLN Travel — waitlist page upgrade

This is the operator playbook. Five steps. ~10 minutes.

---

## 1 · Paste the new page body into Shopify

1. Open https://admin.shopify.com/store/bln-travel/pages/125682286659
2. The body editor has two modes — switch to **`< >` Show HTML**.
3. **Select all** existing content, delete.
4. Open `pages-list-body.html` in this folder, copy the entire file, paste.
5. Click **Save**.
6. Click **View page** at the top to confirm `/pages/list` renders.

You should see: the cream background, "Join the list." heading in Georgia,
a live countdown ticking down toward June 13, the email field, the SMS opt-in
checkbox, and the dark submit button.

> No Klaviyo `<script>` tag is required — the form posts directly to Klaviyo's
> client-side subscriptions endpoint with your company id `SRUrAp`. This avoids
> the Shopify→Klaviyo sync delay you'd get from native customer-tags routing.

---

## 2 · Test a real signup (90 seconds)

1. In an incognito window, open `https://bln-travel.myshopify.com/pages/list?utm_source=test&utm_campaign=setup`
2. Enter your real email, leave SMS unchecked, hit **Join the list**.
3. You should see "You're in. Check your inbox." within 2 seconds.
4. Open Klaviyo → Audience → Lists → **Email List (XHpxSX)** → confirm your
   email appears at the top with these profile properties:
   - `signup_source` = `BLN Waitlist Page`
   - `utm_source` = `test`
   - `utm_campaign` = `setup`
   - `landing_url` = the URL with utm params
5. Repeat with SMS ticked and a real phone number (E.164 like `+15145550123`).
   Confirm the profile lands in **Text Messaging List (VUXEnu)** too.

---

## 3 · Build the popup form in Klaviyo (optional — already inline in the page)

The current page ships the popup **inline** — same Klaviyo `/client/subscriptions`
endpoint, fires at 30 s OR 50 % scroll, suppressed by localStorage for repeat
visitors. You don't need to create anything in Klaviyo's form builder for the
popup to work.

If you want to swap to a Klaviyo-managed form later (so a non-engineer can edit
the popup copy from the Klaviyo dashboard):

1. Klaviyo → Sign-up forms → **Create form** → "Embed" type, named "BLN Waitlist — Popup"
2. Single email field, double opt-in **OFF**, success message: "You're in. Check your inbox."
3. Targeting tab → "Show form on" `pages/list` only, "Display after" 30 seconds OR 50 % scroll
4. Once published, Klaviyo gives you a form ID like `Wabc12`. To swap:
   - In `pages-list-body.html`, **remove** the entire `<div class="bln-popup-backdrop" id="blnPopup">…</div>` block + its trigger code at the bottom of the script.
   - In Shopify theme settings → **Online Store → Themes → Edit code → `layout/theme.liquid`** → just before `</body>` add:
     ```html
     <script async type="text/javascript" src="//static.klaviyo.com/onsite/js/SRUrAp/klaviyo.js?company_id=SRUrAp"></script>
     ```
   That single script powers all Klaviyo dashboard-managed forms across the store.

For tonight's launch, leave the inline popup as-is. It's faster and doesn't
need a Klaviyo form ID.

---

## 4 · OpenGraph / Twitter cards (for Meta + TikTok ad previews)

Shopify renders `<meta>` tags from the page body, but ad scrapers (Meta, TikTok,
Twitter/X) read them from `<head>`. Two options:

### Option A — Shopify SEO panel (fastest)

1. https://admin.shopify.com/store/bln-travel/pages/125682286659
2. Scroll to **Search engine listing** → **Edit**
3. Set:
   - **Page title**: `BLN Travel — Join the list. Drop June 13.`
   - **Description**: `Three colorways. One hundred pieces. The list opens now.`
4. Save.

This covers `<title>`, `og:title`, and `og:description`. For `og:image` you need
Option B.

### Option B — Theme template (proper)

1. **Online Store → Themes → Edit code**
2. Open `layout/theme.liquid`
3. Inside `<head>`, find the block of meta tags, paste this just below them:
   ```liquid
   {%- if page.handle == 'list' -%}
     <meta property="og:type"            content="website">
     <meta property="og:title"           content="BLN Travel — Join the list. Drop June 13.">
     <meta property="og:description"     content="Three colorways. One hundred pieces. The list opens now.">
     <meta property="og:image"           content="https://cdn.shopify.com/s/files/1/{{ shop.id }}/products/champagne-gold-hero.jpg">
     <meta property="og:url"             content="{{ shop.url }}/pages/list">
     <meta name="twitter:card"           content="summary_large_image">
     <meta name="twitter:title"          content="BLN Travel — Join the list. Drop June 13.">
     <meta name="twitter:description"    content="Three colorways. One hundred pieces. The list opens now.">
     <meta name="twitter:image"          content="https://cdn.shopify.com/s/files/1/{{ shop.id }}/products/champagne-gold-hero.jpg">
   {%- endif -%}
   ```
4. Replace the `og:image` URL with the **actual** Champagne Gold featured image
   URL. To find it: Klaviyo or Shopify → product `7666466717763` → main image →
   right-click → Copy image address → paste over the placeholder.
5. Save.
6. Validate at https://developers.facebook.com/tools/debug/ — paste
   `https://bln-travel.myshopify.com/pages/list` → click **Scrape Again**.

---

## 5 · Verify mobile (1 minute on your iPhone)

1. Open `https://bln-travel.myshopify.com/pages/list` on Safari iOS.
2. Tap the email field — the keyboard should appear WITHOUT the page zooming
   in. (CSS guarantees this: `font-size: 16px` on all inputs.)
3. The submit button + SMS checkbox row should be comfortable to tap with a
   thumb — both ≥44 pt tall by design.
4. Tick SMS → the phone field appears smoothly below the email.
5. The countdown ticker should be legible without squinting.
6. Scroll to ~halfway down — the popup should appear (if you haven't already
   submitted on this device).

---

## Reporting back

Once you've completed steps 1–4, ping me with:

- **a.** Body length after paste (Shopify shows the character count in the editor footer)
- **b.** Klaviyo form ID used → **none required** in current build (custom inline form using `/client/subscriptions`). If you swap to dashboard popup later, paste the new form ID here.
- **c.** Any errors from Klaviyo or Shopify → most likely is a 401 from `/client/subscriptions` if the company id is wrong. Open browser DevTools → Network → filter for `klaviyo.com` → check the response.

---

## What's intentionally NOT in this build

- **No jQuery / React / framework code** — vanilla JS only, as you asked.
- **No emoji, no exclamation marks, no "exclusive offer" language** — confirmed.
- **No Klaviyo `<script>` tag on the page** — the custom form is faster and doesn't pull in 200 kB of Klaviyo onsite code. If you later want analytics/identify
  events (e.g., `Viewed Product` tracking from `pages/list`), add the snippet in
  step 3 to `theme.liquid`.
- **No chatbot, no exit-intent popup, no early popup** — popup waits the full 30 s or 50 % scroll, whichever first.
- **No edits to the 13 Klaviyo email templates** — they're untouched.
- **No change to the page handle** — `/pages/list` stays the same so your Meta /
  TikTok ads keep working.
