# Edd.AI 🏠

**Your move-in co-pilot.** Film your rented flat. Get an evidence-grade report that protects your deposit.

Built for the **LBS AI Cup 2026** (Track A – Productivity & Work).

**Live app:** https://movein-copilot.streamlit.app

> Personal copy of our team repo ([edd-ai-lbs/aicup-build](https://github.com/edd-ai-lbs/aicup-build)), with the full commit history.

---

## The problem

Students and international renters in the UK often lose deposit money for damage they didn't cause. They don't know the rules, and a proper inventory takes hours. Edd.AI turns a one-minute phone video per room into a dated, photo-backed report, and then helps at every step until the deposit comes back.

## What it does

The app opens on a short homepage, with tabs across the top for each tool.

### 🏠 Move-in report
A step bar (Film → Check → Report) shows where you are.

1. **Film a room** (or the whole flat) and talk through anything you notice.
2. **The AI finds visible damage.** Claude looks at the key moments of the video and lists each issue with the room, item, problem, severity and the exact second it appears, and **outlines where the damage is** in the photo.
3. **It listens too.** Your spoken comments (via ElevenLabs speech-to-text) go into the right issue's note. Problems a picture can't show ("this window doesn't work") are added as issues.
4. **Check the issues.** Each issue is a card, grouped by room, with its outlined photo, a **Keep in report** switch, and quick edits (severity, who sorts it, your note). It also shows an **AI check** (🟢 sure / 🟠 "check this", with the reason), a **rewatch** button that plays that moment, and **who sorts it** (landlord / tenant / not the tenant's), with the **GOV.UK or Shelter source**. Prefer a spreadsheet? **Edit as table** switches to a full table.
5. **Forgot something?** Record a voice note, and the AI adds new problems or comments on existing ones.
6. **Compare with your agency's inventory.** Upload their PDF and see what it leaves out (raise these!), what it mentions that you didn't film, and get a ready-to-send reply to the agent.
7. **Download the PDF report**, with your address, the date, an outlined photo and timestamp for every issue, and your notes.

### 📦 Move-out check
Film each room again when you leave. The AI compares **move-in vs move-out** side by side:
🔴 new since move-in · 🧽 clean before you leave · 🛡️ already there at move-in (you shouldn't pay) · ✅ fixed. You also get a "to do before you leave" list.

### ✅ Admin checklist
A personalised move-in to-do list for England (deposit protection, right to rent, council tax, energy, TV licence, GP, voting), with GOV.UK and NHS links, based on four quick questions.

> General guidance, not legal advice. Hard cases: [Shelter](https://england.shelter.org.uk/housing_advice) or [Citizens Advice](https://www.citizensadvice.org.uk/housing/renting-privately/).

## How the AI works

```
phone video ──► pipeline/media.py   ffmpeg: sound only (~0.4 MB) + keyframes only (≈1 per second)
                 │
                 ├─► pipeline/voice.py    ElevenLabs speech-to-text → sentences with timestamps
                 └─► pipeline/frames.py   drops blurry, dark and repeated moments → ≤30 pictures
                          │
                          ▼
                 pipeline/vision.py   Claude (one request, all pictures + transcript)
                                       → issues in a shared JSON format, with confidence, a box
                                         around the damage, and a responsibility rule from
                                         content/responsibility.json
```

| Module | What it does |
|---|---|
| `pipeline/analyse.py` | `analyse_room(video, room)`: the one call the app makes; saves results for a demo fallback |
| `pipeline/vision.py` | Damage detection with strict "not a false alarm" rules (blur, glare, reflections, grout lines aren't damage), plus a box around each problem |
| `pipeline/voice_note.py` | "Forgot something?" voice notes → new issues or comments |
| `pipeline/inventory.py` | Agency inventory PDF (text via pypdf) vs the video findings → disagreements + draft reply |
| `pipeline/moveout.py` | Move-in vs move-out video → before/after findings |
| `content/` | Checklist and landlord/tenant rules, each with its GOV.UK / Shelter / NHS source (checked 14 Nov 2026) |
| `report/make_report.py` | The PDF report |
| `app/main.py` | App entry: Edd.AI theme, logo and the tabs across the top |
| `app/home.py`, `app/move_in.py`, `app/pages/` | The screens: homepage, move-in report, move-out check, admin checklist |
| `app/ui.py` | Edd.AI look: colours, fonts, cards, badges, step bar, animations |

**Speed and cost** (65 s, 500 MB 4K phone video): about **16 s** end to end and about **$0.15** of Claude usage.

## Run it locally

You need Python 3.12 and two API keys: [Anthropic](https://console.anthropic.com) (Claude) and [ElevenLabs](https://elevenlabs.io) (speech-to-text only).

```bash
git clone https://github.com/griziottifrancesca/edd-ai.git
cd edd-ai
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

Create a file called `.env` in the project folder (it's in `.gitignore`, so it is never uploaded):

```
ANTHROPIC_API_KEY=sk-ant-...
ELEVENLABS_API_KEY=sk_...
```

Then start the app and open http://localhost:8501:

```bash
.venv/bin/python -m streamlit run app/main.py
```

**Deploying on Streamlit Community Cloud:** main file `app/main.py`, Python 3.12, and the two keys in the app's **Secrets** box (TOML format, with quotes: `ANTHROPIC_API_KEY = "sk-ant-..."`).

## Data and privacy

- Videos, pictures and inventories stay in `data/`, which `.gitignore` keeps out of the repo (only the pre-saved demo results in `data/demo_cache/` are committed).
- API keys live only in `.env` / Streamlit Secrets.
- Only our own videos, consented videos from classmates, and public GOV.UK / Shelter / NHS guidance were used.

## Team

We built Edd.AI in about 20 hours, over one weekend, as a team of three.

### Anant – technical lead
- Set up the repo and checked every merge into `main`
- Led the Edd.AI rebrand and redesign: homepage, top tabs, green theme, issue cards
- Outlined the damage on photos, in the app and in the PDF
- AI-written landlord email
- Move-out PDF report
- Demo fallback (saved results for the demo videos)

### Emma – app and report
- First working app: upload a room video, edit the list of issues, download the PDF
- Voice notes on the app screen
- "AI check" column and rewatching the moment in the video
- Who sorts each problem, with its source
- Admin checklist page
- Agency inventory comparison screen
- Move-out page

### Francesca – AI pipeline, content and pitch
- AI pipeline: splits the video into sound and pictures and keeps only the sharp, useful frames
- Speech-to-text with ElevenLabs
- Damage detection with Claude: strict "no false alarm" rules, a confidence level, and a source for who is responsible
- Voice-note, agency-inventory and move-out comparison logic
- Speed-ups: a 500 MB 4K video in about 16 s
- Phone-friendly layout
- UK rules and checklist content (GOV.UK, Shelter, NHS)
- This README, the pitch and the demo script

Built with [Claude](https://www.anthropic.com/claude), [ElevenLabs](https://elevenlabs.io), [Streamlit](https://streamlit.io), OpenCV, ffmpeg, fpdf2 and pypdf.
