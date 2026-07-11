# SIR गणना प्रपत्र सहाय्यक — Static Site

A single-file static website that lets anyone fill the Maharashtra SIR enumeration form (गणना प्रपत्र), take a photo whose background is automatically turned white, pay ₹10 via UPI QR, and download a print-ready reference PDF. Everything runs in the visitor's browser — no server, no database, no data ever leaves their device.

## Deploy on GitHub Pages (5 minutes)

1. Create a new **public** repository on GitHub, e.g. `sir-form-seva`.
2. Upload `index.html` to the repository root (Add file → Upload files → Commit).
3. Go to **Settings → Pages → Source: Deploy from a branch → Branch: main / (root) → Save**.
4. In 1–2 minutes your site is live at `https://<your-username>.github.io/sir-form-seva/`.

## Configure before going live (REQUIRED)

Open `index.html`, find the `CONFIG` block near the bottom:

```js
const CONFIG = {
  UPI_VPA : "yourname@upi",     // ← replace with YOUR UPI ID (e.g. 9876543210@ybl)
  UPI_NAME: "SIR Form Seva",    // shown in the payer's UPI app
  AMOUNT  : "10",
  REQUIRE_PAYMENT: true         // set false to make payment optional
};
```

The QR code and the "open UPI app" link are generated from these values automatically.

## How each feature works

| Feature | How | Notes |
|---|---|---|
| Form filling | Plain HTML inputs, Marathi-first labels | Live mirror: the paper-form preview fills in ballpoint-blue as the user types |
| Photo → white background | MediaPipe Selfie Segmentation (runs in-browser via CDN, ~2 MB model) | If the model fails to load, the original photo is used and the user is warned |
| PDF | html2pdf.js (html2canvas + jsPDF) rasterizes the preview | Devanagari renders perfectly because the browser does the text shaping |
| UPI ₹10 | `upi://pay` deep link rendered as QR (qrcodejs) | See honesty note below |
| Privacy | Zero network calls with user data | Only CDN library/font downloads occur |

## ⚠️ Honest limitations you should know

1. **Payment cannot be enforced on a static site.** There is no backend to verify UPI transactions. The site asks for the last 4 digits of the UTR + a confirmation checkbox — friction, not proof. Anyone can bypass it via browser dev tools. If you need real enforcement, the upgrade path is:
   - **Razorpay Payment Button** (free to embed on static sites; you verify manually in dashboard), or
   - a tiny backend (Cloudflare Workers free tier) + Razorpay/Cashfree webhook that issues a one-time unlock code.
2. **The generated PDF is a REFERENCE COPY.** The BLO-issued enumeration form carries a voter-specific QR code; officials expect that physical form back. Users must copy the values onto the original — the PDF makes that a 2-minute job and is also a record for their file. The PDF prints this disclaimer at the top; do not remove it.
3. **Charging for form assistance** is legal (cyber-café model), but do not present the service as official/ECI-affiliated. The footer disclaimer covers this — keep it.
4. **First photo takes a few seconds** on slow connections (model download). Subsequent photos are instant.

## Customization ideas

- Pre-fill district/AC defaults for your area (edit `value=""` attributes).
- Add your WhatsApp support number in the footer.
- Hindi toggle: duplicate labels with a `lang` switcher.

## Tech stack (all via CDN, nothing to build)

- html2pdf.js 0.10.1 (cdnjs)
- qrcodejs 1.0.0 (cdnjs)
- @mediapipe/selfie_segmentation (jsdelivr)
- Google Fonts: Tiro Devanagari Marathi + Noto Sans Devanagari
