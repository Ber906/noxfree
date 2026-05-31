# ADS_SETUP_GUIDE.md — Noxfree Rewarded Ads Setup

How to replace the placeholder `playRewardAd()` function in `index.html` with a real ad.
Look for the comment `// REPLACE THIS WITH YOUR REAL AD CODE - SEE GUIDE BELOW` to find
the exact place to paste your code.

---

## Option A — Google AdSense Rewarded Ads

**Best for:** Long-term revenue, highest CPM. Requires approval and a live domain.

**Steps:**

1. Go to https://adsense.google.com and sign up / log in.
2. Click **"Add site"** and enter your live domain (e.g. `noxfree.yourdomain.com`).
   - ⚠️ AdSense requires a **real custom domain** — localhost won't work.
3. Paste the AdSense auto-ads `<script>` tag inside `<head>` in `index.html`.
4. Wait for site approval (usually 1–14 days).
5. After approval: **Ads → By ad unit → Rewarded ads → Create new ad unit**.
6. Copy the generated Publisher ID: `ca-pub-XXXXXXXXXXXXXXXX` and Ad Unit ID.
7. Add the rewarded ad SDK in `<head>`:
   ```html
   <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXXXXXXXX" crossorigin="anonymous"></script>
   ```
8. Replace the body of `playRewardAd()` in `index.html` with:
   ```javascript
   (adsbygoogle = window.adsbygoogle || []).push({
     googletag: {
       pubads: function() {
         return {
           addEventListener: function(event, cb) {
             if (event === 'rewardedSlotGranted') cb();
           }
         };
       }
     }
   });
   // Use the AdSense Rewarded Ad API call here — follow the docs at:
   // https://support.google.com/adsense/answer/9183460
   ```
   Then call `onAdCompleted()` inside the "reward granted" callback.

**Notes:**
- Do NOT run AdSense on sites with pirated / unlicensed content. You will be banned.
- Test using AdSense test mode first before going live.

---

## Option B — AdinPlay / VAST Rewarded Ads

**Best for:** Faster approval than AdSense, video game / streaming sites welcome.

**Steps:**

1. Sign up at https://adinplay.com
2. **Add Site** → enter your domain → get approved (usually 24–72 hours).
3. Go to **Ad Tags** and create a **VAST Rewarded Video** tag. Copy the VAST URL.
   It looks like: `https://vast.adinplay.com/ads/...`
4. Add a VAST player library to `<head>` in `index.html`:
   ```html
   <script src="https://imasdk.googleapis.com/js/sdkloader/ima3.js"></script>
   ```
5. Replace `playRewardAd()` with:
   ```javascript
   function playRewardAd() {
     var vastUrl = "PASTE_YOUR_VAST_URL_HERE";  // ← paste AdinPlay VAST tag here
     var adContainer = document.createElement("div");
     adContainer.style.cssText = "position:fixed;top:0;left:0;width:100%;height:100%;z-index:99998;background:#000";
     document.body.appendChild(adContainer);
     var adDisplayContainer = new google.ima.AdDisplayContainer(adContainer);
     adDisplayContainer.initialize();
     var adsLoader = new google.ima.AdsLoader(adDisplayContainer);
     adsLoader.addEventListener(google.ima.AdsManagerLoadedEvent.Type.ADS_MANAGER_LOADED, function(e) {
       var adsManager = e.getAdsManager({currentTime: 0, duration: -1});
       adsManager.addEventListener(google.ima.AdEvent.Type.ALL_ADS_COMPLETED, function() {
         document.body.removeChild(adContainer);
         onAdCompleted();  // ← fires "Thanks for supporting" toast
       });
       adsManager.init(640, 360, google.ima.ViewMode.NORMAL);
       adsManager.start();
     });
     var adsRequest = new google.ima.AdsRequest();
     adsRequest.adTagUrl = vastUrl;
     adsLoader.requestAds(adsRequest);
   }
   ```

**Notes:**
- AdinPlay is more lenient about content type than AdSense.
- Still requires a public domain; test on staging before production.

---

## Option C — Monetag / PropellerAds Rewarded Interstitial

**Best for:** Easiest and fastest signup, works on most sites, no strict content rules.

### Monetag

1. Sign up at https://monetag.com
2. **Add site** → get verified.
3. Go to **Ad Units → Rewarded Interstitial** → Create → Copy the script.
4. Paste the Monetag script tag in `<head>` of `index.html`.
5. Replace `playRewardAd()` with the Monetag show-ad call they provide, e.g.:
   ```javascript
   function playRewardAd() {
     // Monetag rewarded interstitial trigger
     // Replace "ZONE_ID" with your actual zone ID from Monetag dashboard
     show_rewarded_8399217 = function() {      // ← Monetag generates this function name
       onAdCompleted();  // fires after the user watches the ad
     };
     // Monetag SDK call (exact name depends on their current snippet):
     window.__rewarded && window.__rewarded.show("ZONE_ID");
   }
   ```

### PropellerAds

1. Sign up at https://propellerads.com
2. **Sites → Add site** → verify ownership.
3. Create a **"Onclick/Popunder"** or **"Interstitial"** ad unit.
4. Copy the script tag and paste into `<head>` of `index.html`.
5. Wire the trigger similarly to the Monetag example above, calling `onAdCompleted()` in the success callback.

**Notes:**
- These networks accept most traffic including streaming / entertainment sites.
- Their CPM is lower than AdSense but approval is nearly instant.

---

## Important Reminders

| ⚠️ | Detail |
|----|--------|
| **Legal content only** | Running ads on sites hosting pirated / unlicensed movies will get your account banned and may expose you to legal risk. |
| **Custom domain required** | AdSense will not approve localhost or `.replit.app` subdomains. Use a real domain (Namecheap, Google Domains, etc.). |
| **Test mode first** | Always enable each network's test/sandbox mode and verify ads load before switching to live. |
| **One network at a time** | Running multiple ad scripts simultaneously can slow your page and cause policy violations. |
| **GDPR / privacy** | If your users are in the EU, you need a consent banner (CMP) before serving personalized ads. |

---

## Where to edit in index.html

Search for this comment in `index.html`:

```
// REPLACE THIS WITH YOUR REAL AD CODE - SEE GUIDE BELOW
```

Replace the body of `playRewardAd()` (just below that comment) with whichever
option above fits your situation. Keep calling `onAdCompleted()` after a successful
ad view — that's what shows the "Thanks for supporting" toast to the user.
