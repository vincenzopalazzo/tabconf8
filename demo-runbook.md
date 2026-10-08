# Live demo runbook: TABConf 8 main stage
**Talk:** “Bolt12 is the new common interface to Bitcoin payments”
**Slot:** Tue 13 Oct 2026, 3:45–4:15 PM ET (12:45–1:15 PM PT), Main stage, Georgia Tech Exhibition Hall, Atlanta
**Demo window:** slides 27 (“Watch.”) and 28 (fallback video), about 4 minutes, starting about 3:59 PM ET. *(Renumbered again in v2.3: a Spark slide was added at 22, so the deck is 40 slides.)*
*Drafted 5 Oct 2026. Items marked **[DECIDE]** need Vincenzo's choice. Items marked **[VERIFY]** are not confirmed from a primary source and must be tested on the device.*

---

## 0. What the audience must see (the one thing to prove)
> **One QR that never changes, paid by different wallets, then paid by a name. No invoice copied, no server asked.**

Three payments to **one offer**:
1. Pay #1: scan the QR and pay with a message.
2. Pay #2: scan the **same** QR from a **different wallet**.
3. Pay #3: type `₿USER@DOMAIN` (BIP-353), which resolves to the same offer.

---

## 1. Props: choose one receiver and one or two payers **[DECIDE]**

### Receiver options
| | **R1: Phoenix receives** (simplest) | **R2: off-site Core Lightning node** (strongest proof) |
|---|---|---|
| What's on screen | The Phoenix payment list fills up on the mirrored phone | A terminal shows N invoices, each with a different `payment_hash`, for **one** `offer_id` |
| Pros | No server. It's the "users never see the plumbing" story itself. One less thing to SSH into. | Venue Wi-Fi can't break the receiver. Proves "one offer, many invoices" visually. Phoenix stays free to be the best payer (BIP-353 and contacts). |
| Cons | The receiver phone is on the venue network or your hotspot. Phoenix needs inbound liquidity, and the first big receive can trigger an LSP channel or splice fee and a prompt. Showing "different invoices" is less visible. | You need a funded node with **inbound** liquidity and onion-message-capable peers, set up days ahead. Extra laptop pane. |
| Offer source | Phoenix's own reusable offer (Receive screen). **[VERIFY]** the exact taps: Phoenix shows BOLT11 by default and has had BOLT12 since v2.3.1. | `lightning-cli offer any "TABConf 8 demo"` → save `bolt12` and `offer_id`. CLN is at v26.06.8 (22 Sep 2026). Pin the version you rehearse with. |

**Recommendation:** use **R2** if the node is ready by Sat 10 Oct. Otherwise use **R1**. Either way, the offer string must be final by **Sat 10 Oct**, because it goes into the deck, the QR and the DNS record.

### Payer options
| Phone | Wallet | Status (checked 5 Oct 2026) | Notes |
|---|---|---|---|
| **A** (with R2) | **Phoenix** (ACINQ) | Pays offers since v2.3.1 and BIP-353 since v2.3.3. Latest is android-v2.8.4 (2 Oct 2026). | Best payer for the BIP-353 step. With R1, Phoenix is the receiver instead, so the payers are B1 and/or B2. |
| **B1** | **Strike** **[DECIDE]** | Pays offers via LNDK since Aug 2024 ([Strike blog](https://strike.me/blog/bolt12-offers/)). | Custodial and the simplest flow. Strike's blog *describes* BIP-353 name entry, but **[VERIFY]** it in the app. Needs a funded, KYC'd account and the US region on your phone. |
| **B2** | **Zeus** **[DECIDE]** | v13.2.2 (23 Sep 2026): embedded LDK Node backend, DNSSEC-enforced BIP-353 BOLT 12 resolution ([release](https://github.com/ZeusLN/zeus/releases/tag/v13.2.2)) | Self-custodial ("different company, self-custody" beat). The embedded node needs a channel and outbound liquidity days ahead, and it must be online and synced before you walk on. |

Suggested pairings:
- **R2 + Phoenix (A) + Strike or Zeus (B):** the original plan. Phoenix does pays #1 and #3 (BIP-353), and B does pay #2.
- **R1 (Phoenix receives) + Zeus (B2) for pays #1 and #3 + Strike (B1) for pay #2.** This needs three phones, or drop Strike and have Zeus pay twice (weaker "different wallet" beat).

### Optional "Receipts" beat (only if there's time, about 20 s)
- bolts#1346 (payer proofs) **was merged 27 Jul 2026**. Your Lexe tooling returns a payer proof (`lnp1…`) plus an lnproof.space URL after a BOLT 12 pay. That could be a one-line flash: "and here's my receipt." **The Lexe MCP connection on the box was down on 5 Oct** (connection refused on 127.0.0.1:8010), so this is untested. Skip it unless you rehearse it.

### Hardware
- Laptop running the deck (`deck.html` in Chrome, full screen; `n` toggles notes) with the **local MP4** for slide 28.
- The receiver phone or payer phone(s), each with a USB cable for mirroring (Android: `scrcpy`; iPhone: QuickTime → New Movie Recording → choose the iPhone).
- A personal hotspot on a **different carrier** from the venue Wi-Fi, plus a USB-C hub and HDMI adapter.
- Printed A5 card with the offer QR and `₿USER@DOMAIN` (backup prop, and a handout for Q&A).
- USB stick with: deck.html, deck.pdf, deck.pptx, demo.mp4.

---

## 2. BIP-353 name setup (domain = `DOMAIN`, user = `USER`) **[DECIDE]** the domain
1. The domain must be **DNSSEC-signed** (BIP-353 requires it). Check: `dig +short DS DOMAIN` (it needs a DS record at the parent) and `delv @1.1.1.1 DOMAIN SOA` (it must say "fully validated").
2. Add a TXT record:
   - Name: `USER.user._bitcoin-payment.DOMAIN`
   - Value: `bitcoin:?lno=<FULL_LNO1_OFFER>`
   - Offers are long. If the DNS host limits one TXT string to 255 characters, split the value into several quoted strings in the **same** record (resolvers concatenate them). Don't create several TXT records.
   - TTL: 300 s while testing. Leave it alone after Sat 10 Oct.
3. Verify from **a phone on mobile data**, not just the laptop:
   - `delv @1.1.1.1 TXT USER.user._bitcoin-payment.DOMAIN` → "fully validated", and the value starts with `bitcoin:?lno=lno1`.
   - Resolve it in each payer wallet you will use on stage (Phoenix / Zeus / Strike **[VERIFY]**). If you use R2, also run `lightning-cli xpay ₿USER@DOMAIN 1000sat`, or the equivalent for your CLN version.
4. The deck: put `₿USER@DOMAIN` on slide 40 (closing) and on the printed card.

---

## 3. On-stage click path (target 4:00, hard stop 5:00)
Slide 27 "Watch." is up. Mirror window on the left, terminal on the right (R2) or the receiver phone mirror (R1).

| Clock | Say | Do (Phone / screen) | Success signal |
|---|---|---|---|
| 0:00–0:20 | "This is one QR. It's printed on this card too. It never changes." | Hold up the card. The QR is on the screen (the "lno1…" slide or the mirror). | — |
| 0:20–1:20 | "Pay #1, from Phoenix." *(R1: "from Zeus")* | **Payer:** open the wallet → **Scan** → point at the QR on the big screen or the card → amount **1,000 sat** → message **"hello TABConf"** → **Pay/Send** → wait. | Payer shows "Sent". R2: the terminal row flips to `paid` (invoice #1). R1: Phoenix shows +1,000 sat with the note. |
| 1:20–2:20 | "Different wallet. Different company. **Same QR.**" | **Second payer (Strike or Zeus):** **Scan** the same QR → **1,000 sat** → **Pay**. | Invoice #2 is paid, with a **different payment_hash** (R2: point at it). |
| 2:20–3:20 | "Now nobody scans anything. You just say my name." | **Payer (Phoenix or Zeus):** **Send** → type `₿USER@DOMAIN` (or `USER@DOMAIN`) → the wallet shows the resolved offer → **1,000 sat** → **Pay**. | Invoice #3 is paid, from the same `offer_id`. |
| 3:20–4:00 | "No invoice was copied. No server was asked. Three payments, one offer. **The plumbing stayed hidden.**" | Back to the deck. Skip slide 28 and go to slide 29. | — |

Optional, only if ahead of time (Android Phoenix): save the offer as a **contact** and pay with one tap (bLIP-42), about 30 s.

### R2 terminal pane (prepare before going on)
```bash
ssh demo-node
watch -n1 'lightning-cli listinvoices | jq -r ".invoices[] | select(.local_offer_id==\"OFFER_ID\") | [.status, (.amount_received_msat//0), .payment_hash[0:16]] | @tsv"'
```
Use a font of 28 pt or larger and a dark theme. **[VERIFY]** the field name (`local_offer_id`) on your CLN version. Rehearse with exactly this command.

---

## 4. Failure modes and when to switch to the recording
**Abort rule:** if any single step goes past **45 s**, or **two** steps fail, or the mirror isn't working at 0:00, say the line and play slide 28:
> "The network's fine. The conference Wi-Fi isn't. Here's the same thing from yesterday."

If pays #1 and #2 worked and only BIP-353 fails, play only the **BIP-353 chapter** of the video (keep a chapter timestamp in the notes), then move on. Don't retry on stage more than once.

| Symptom | Likely cause | 10-second fix on stage | If not fixed |
|---|---|---|---|
| Payer spins on "fetching invoice" / timeout | The invoice_request onion message isn't reaching the receiver (peer offline, no onion-message path) | Retry **once** | Video |
| "No route" / insufficient liquidity | Receiver inbound or payer outbound liquidity used up | Lower the amount to 500 sat and retry once | Video. Check the liquidity margin the next day. |
| Name doesn't resolve / "invalid address" | DNSSEC chain broken, the resolver is filtering, or the wallet version lacks BIP-353 | Say "the same name, as a QR" and scan the card | BIP-353 chapter of the video |
| Phoenix (R1) shows a fee or channel prompt | Inbound liquidity was exhausted, so an LSP channel or splice is needed | Accept if it's small | Video. Pre-receive more beforehand. |
| Strike asks for re-login, 2FA or a limit | Session expired, region, or KYC limit | — | Use the other payer, or the video |
| Zeus embedded node not synced | The app was backgrounded or killed | Wait at most 20 s with the app open | Use the other payer, or the video |
| Mirror freezes or the HDMI drops | Cable or AV problem | Re-plug once | Video (it lives on the laptop) |
| Venue Wi-Fi dead | Venue | The phones are already on mobile data or the hotspot | Video |

---

## 5. Recorded backup (do this on Mon 12 or Tue 13 Oct, at the venue)
- Record the **exact** stage run (steps 0:00–4:00) as a 1080p screen capture of the mirror plus the terminal. No audio dependency, and no live narration needed.
- Export it as `demo.mp4` (H.264), about 3 min, with chapter markers noted: pay #1 = mm:ss, pay #2 = mm:ss, BIP-353 = mm:ss.
- Embed it in slide 28 as a **local file** and test playback from the deck in full screen. Put one copy on the USB stick and one on the phone.

---

## 5b. "Same name, different backend" (supports slides 20–25)
**Verdict: show it on slides on stage. Optionally add a pre-recorded 40-second clip. Don't swap live.**

Why there's no live swap:
- **The swap is a DNS change, not a wallet action.** The name stays the same, but the TXT record must switch to the new backend's offer. DNSSEC re-signing at your DNS host, resolver caches (TTL) and wallet-side caching (Phoenix contacts, Zeus) make the timing unpredictable. That's a guaranteed 45-second breach.
- **Today the swap only works between Lightning backends** (Phoenix ↔ CLN ↔ LDK Node). A Spark or Arkade receiver can't sit behind an offer yet: the Spark SDK and Arkade docs only document BOLT11. So a live "→ Spark" swap isn't possible. Say so; it's the call to action.

### Option S1 (default): slides only
- Slides 24 → 25 carry it. If you want a diagram, put one under slide 25 (optional, built in Keynote or Slides):
  ```
  ₿USER@DOMAIN ──DNS──▶ lno1…(A)  ──▶ Phoenix
                    └─(swap)─▶ lno1…(B)  ──▶ CLN node    [later: Spark / Ark receiver]
  payer types the SAME name both times · observers see "an offer", not which layer
  ```
- Line: "I moved my name from Phoenix to my own node last week. Nobody who pays me had to change anything."

### Option S2 (optional): pre-recorded swap clip, about 40 s, as a chapter of `demo.mp4`
Record on **Mon 12 Oct**, after the main demo recording:
1. TXT `USER.user._bitcoin-payment.DOMAIN` → `bitcoin:?lno=<OFFER_A>` (Phoenix, receiver R1). In the terminal, run `delv TXT …` and show `lno1…A`.
2. Payer: type `₿USER@DOMAIN` and pay 1,000 sat. Phoenix (A) receives it.
3. Update the TXT record to `bitcoin:?lno=<OFFER_B>` (CLN, receiver R2). Use TTL = 60 s and set it a few minutes before recording. Wait until `delv` shows `lno1…B` as "fully validated".
4. Payer: clear any saved contact or cache, type the **same** `₿USER@DOMAIN`, and pay. The CLN terminal shows it paid.
5. Caption: "Same name. Different backend. The payer never knew."
6. **Set the TXT record back** to whatever the live stage demo uses, then re-verify with `delv` on Tue 13 Oct and the morning of Tue 13 Oct.
- Play it right after slide 25, or inside the demo window if time allows. If any step misbehaves during recording, drop S2. It's a nice-to-have.

## 6. Setup timeline
- **By Sat 10 Oct (before flying):** receiver chosen (R1 or R2). Offer created and final. QR rendered (`qrencode -s 20 -o offer.png "<lno1…>"`) and placed on slides 15, 27 and 40. Printed card. DNS TXT record live and validated. Payer wallets funded (at least 20k sat each), and the second payer chosen. Full rehearsal #1 at home.
- **Mon 12 Oct:** AV check with the main-stage crew (HDMI, resolution, whether they can take a second input). Rehearsal #2 on venue Wi-Fi. Record the backup video.
- **Tue 13 Oct:** rehearsal #3 **hotspot only**. Fix anything that took more than 30 s.

## 7. Pre-stage checklist: morning of Tue 13 Oct (all times ET, Atlanta)
- [ ] **8:00:** receiver healthy. R2: `lightning-cli getinfo` and `listpeerchannels` (peers connected, inbound ≥ 10× the demo total). R1: Phoenix opened and synced, with an inbound or fee-free receive margin.
- [ ] **8:15:** BIP-353 check from a phone on mobile data: `delv` gives "fully validated", and each payer wallet resolves `₿USER@DOMAIN`.
- [ ] **8:30:** one real end-to-end test payment from **each** payer to the offer, over the hotspot.
- [ ] **8:45:** check Boltz status (slide 12 wording) and skim for any Spark or Ark Bolt12 news (slides 31–32).
- [ ] **9:00:** wallets: auto-update **off**, app versions written down, batteries above 90%, Do Not Disturb **on**, screen lock set to 10 min or "never", font size large, dark mode.
- [ ] **10:30:** at the stage: laptop on power, deck open on slide 1, mirror windows arranged, terminal pane ready (R2), and `demo.mp4` test-played once.
- [ ] **11:30:** hotspot on and both phones connected, or on mobile data. **Turn venue Wi-Fi off on the phones.**
- [ ] **11:45:** apps open on their Scan / Send screens, the printed card in your pocket, water.
- [ ] **11:55:** last look: the receiver still has inbound liquidity, and the Zeus node (if used) is synced.

---

## 8. Open decisions for Vincenzo
1. **Receiver:** R1 Phoenix receive, or R2 off-site CLN?
2. **Second payer:** Strike (B1) or Zeus (B2)? Or both?
3. **DOMAIN and USER** for BIP-353, and does that domain already have DNSSEC?
4. **Receipts flash with a payer proof** (Lexe `lnp1…`)? Yes or no. The Lexe MCP needs fixing first.
5. Is a third phone available, or is there a second person to hold a phone?
6. Add the pre-recorded "same name, different backend" clip (S2), or stay slides-only (S1)?


## Public explorer references (keep + reproduce)

Saved 5 Oct 2026 for the privacy beat (BIP-353 with layer address vs Bolt12 offer).

| Layer | Explorer URL | What it shows |
|---|---|---|
| Arkade | https://arkade.space/tx/727c9d9375dd40bc7530841d149fc1ccb3bf6de322f790ed3ea2538bfea29e4c | Settled Arkade tx, 14 May 2026, 0.00347859 BTC → `ark1qzpq904a…euwezqkp4yac` |
| Spark | https://sparkscan.io/tx/bc097da104bc876be2d8d1e5d1156b791f2ecbc5da19873287b117588804fefd?network=mainnet | Confirmed USDB token transfer ~$79, 25 Mar 2026, `spark1…` from/to |

### Reproduce for the talk
1. On stage or in the backup video: open both links (or fresh txs Vincenzo creates) so the room sees activity is publicly browsable without login.
2. Preferred: Vincenzo sends a tiny Arkade payment and a tiny Spark transfer from wallets he controls, then shows the new explorer pages (fresher than the reference txs).
3. Contrast beat: same BIP-353 name resolving to `lno1…` — no arkade.space / sparkscan.io trail for the payment.
4. Do **not** publish his personal recurring receive addresses into BIP-353; only use throwaway demo amounts.
