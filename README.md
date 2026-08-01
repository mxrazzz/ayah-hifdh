# Ḥifdh & Murājaʿah

A private, offline-capable app for memorising Qur'ānic proofs (adillah) across the Islamic sciences,
then keeping them alive through spaced revision and testing.

**188 ayāt · 12 sciences · 71 topics · 385 tafsīr notes · 172 ḥadīth references**

---

## Files in this repo

| File | What it is |
|---|---|
| `index.html` | The whole app — UI, ḥifdh engine, spaced repetition, quiz, password gate |
| `data.enc` | The ayah library, AES-256-GCM encrypted. Useless without the password |
| `build.html` | Local tool to re-encrypt `data.enc` after you edit the source |
| `sw.js` | Service worker — offline support, network-first so deploys land immediately |
| `manifest.json` | Makes it installable as an app on your phone |
| `.gitignore` | Keeps `ayat.js` (the readable source) out of the public repo |

**`ayat.js` is deliberately NOT in this repo.** Keep it on your own machine and in a backup.
It is the only readable copy of the library.

---

## Publishing

Already public at GitHub Pages. Any file you commit goes live within about a minute —
no build step, no action to take. The service worker is network-first, so your phone
picks up the new version the next time you open it.

To install on a phone: open the site → Share → **Add to Home Screen**.

---

## The password

The library is encrypted with **AES-256-GCM**, key derived by **PBKDF2-SHA256 at 310,000 iterations**.
A wrong password fails GCM authentication — the app cannot show partial or garbled data, it simply
refuses. There is no hidden copy of the plaintext anywhere in the repo.

Once you enter it correctly, the derived key (not the password) is stored on that device for 90 days
so you are not typing it every time. **Settings → Lock the app** clears it.

If you forget the password AND lose your copy of `ayat.js`, the data is gone. Nobody can recover it.

---

## Adding ayāt later

1. Open `ayat.js` in any text editor. Copy an existing entry and edit it:

```js
{
  id: "aq-ul-10",                     // must be unique
  sci: "aqeedah",                     // a key from SCIENCES
  topic: "Tawḥīd al-Ulūhiyyah",       // free text; groups the card
  tier: 1,                            // 1 = Asās (foundation), 2 = Bināʾ
  q: "Someone asks … what is the daleel?",   // must end with "?"
  ref: "Sūrah name 12:34",
  key: "12:34",                       // powers the quran.com tafsīr link
  ar: "…",                            // Uthmānī text — copy exactly from quran.com
  en: "…",                            // Muḥsin Khān & Hilālī
  t: [ ["Ibn Kathīr", "…"], ["Al-Saʿdī", "…"] ],          // tafsīr notes
  h: [ ["Ṣaḥīḥ al-Bukhārī", "…", "search terms"] ]        // optional ḥadīth
}
```

Get accurate Arabic and translation from:
`https://api.quran.com/api/v4/verses/by_key/12:34?translations=203&fields=text_uthmani`
(translation `203` is Muḥsin Khān & Hilālī). Copy the `text_uthmani` value exactly.

2. **To add a whole new science**, add an entry to `SCIENCES` at the top of the file:

```js
seerah: { name: "Sīrah", ar: "السيرة", c: "#c98fd6" },
```

`c` is the colour used for its dot and tags. Then give ayāt `sci: "seerah"`.

3. **To add a mission pack**, add to `MISSION`:

```js
ramadan: { name:"Ramaḍān Pack", ar:"رمضان", c:"#5fc4b8",
           desc:"One line describing it.", ids:["fq-6","fq-13","qr-1"] }
```

4. Open `build.html` in your browser → pick `ayat.js` → enter the password → it validates
   everything (duplicate ids, missing fields, bad verse keys, missing Arabic, mission packs
   pointing at ids that don't exist) and downloads a fresh `data.enc`.

5. Drag the new `data.enc` into the GitHub repo → Commit. Done.

Your ḥifdh progress is keyed by ayah `id` and lives in your browser, so adding new ayāt never
disturbs what you have already memorised — as long as you don't change existing ids.

---

## How the two systems work

**Ḥifdh** — four stages per ayah:
1. **Read** — question, ayah, translation, tafsīr, ḥadīth
2. **Trace** — the ayah with words blanked; tap a blank to check (density adjustable)
3. **Recall** — question only; recite, then reveal
4. **Lock in** — say the ayah *and* its reference, then mark it memorised

Locking it in moves the ayah into Murājaʿah with a 1-day first interval.

**Murājaʿah** — only holds what you have memorised:
- **Revise** — question → ayah → translation → rate Again / Good / Easy.
  Intervals: 1 · 3 · 7 · 16 · 35 · 75 days. "Again" resets and re-queues it in the same session.
- **Test me** — multiple choice, alternating between *"which ayah would you use?"* and
  *"which question does this ayah answer?"*. **A wrong answer resets that ayah to due today**,
  so the test feeds the schedule rather than sitting beside it.

---

## Sources

- **Arabic**: Uthmānī script from the quran.com API (`text_uthmani`), trimmed to the relevant phrase.
- **Translation**: Muḥsin Khān & Hilālī.
- **Tafsīr notes**: condensed and attributed — Ibn Kathīr, aṭ-Ṭabarī, al-Saʿdī, Ibn Taymiyyah,
  Ibn al-Qayyim, Ibn ʿUthaymīn, ash-Shāfiʿī, Imām Mālik, ash-Shāṭibī, al-Khaṭīb al-Baghdādī,
  Ibn ʿAbd al-Barr, and statements of the ṣaḥābah. Every card links to the full tafsīr on quran.com.
- **Ḥadīth**: given by wording and collection rather than by number, each linking to a sunnah.com
  search — deliberately, so you land on the real text instead of trusting a number.

**These notes are summaries, not verbatim quotations.** Verify against the linked source before
using anything in daʿwah or from a minbar.
