# Skill-Swap

# SkillSwap: find teammates by skill

Students have startup ideas but not the people to build them. SkillSwap lets you post an idea with the skills you need, ranks the people who fill the *missing* skills, lets you chat, and builds the team.

## Run it (5 minutes)
```bash
python -m venv venv
venv\Scripts\activate          # Mac/Linux: source venv/bin/activate
pip install -r requirements.txt
copy .env.example .env         # Mac/Linux: cp .env.example .env
python seed.py                 # optional demo users (password: demo1234)
python app.py                  # open http://127.0.0.1:5000
```
Groq is optional. For AI features, get a free key at console.groq.com and put it in `.env` as `GROQ_API_KEY=...`. Without a key everything still works using built-in fallbacks.

## AI troubleshooting
Open **http://127.0.0.1:5000/ai-check**. It shows whether the key was found, which file it came from, and the exact error from Groq. Common fixes: the file must be named `.env` (not `.env.txt`), no quotes or spaces around the key, and restart `python app.py` after editing it.

## Themes and messaging
- **Light / dark theme:** the round button in the top bar switches themes and remembers your choice.
- **Live messages:** chat refreshes every 2.5 seconds, shows sent and seen ticks, and times are shown in your own timezone. On every page a pop-up appears when a new message arrives and the Messages badge updates every 5 seconds. This uses polling, not WebSockets, which keeps the code simple to explain.

## Groq models
Groq retired the Llama 3.x models on 16 Aug 2026. The app now tries `openai/gpt-oss-20b`, `openai/gpt-oss-120b` and `qwen/qwen3.6-27b`, then any chat model your key lists. To force one, set `GROQ_MODEL=...` in `.env`. Open `/ai-check` to see every attempt.

## Features
- **Tech stack suggestion:** on the Post an idea page, the AI recommends a frontend, backend, database, hosting and first steps, and fills in the skills you need. It is saved on the project page and can be regenerated.
- Sign up and log in (hashed passwords, CSRF protection)
- Profile with skills, skill level and weekly hours
- Post a project with required skills; "Suggest from my pitch" uses Groq
- Ranked matches that only count skills the team is still missing
- In-app chat (auto-refreshes every 4 seconds), AI-drafted opening message
- Invite people to the team; they accept or decline; the team sheet updates
- "Learn from similar startups that closed": 3 matches from `data/failed_startups.csv`, with Groq rewriting the lessons for your idea

## Files
| File | Purpose |
|---|---|
| `app.py` | All backend logic: routes, matching, Groq helpers |
| `templates/`, `static/` | Pages, styling, small JS |
| `data/failed_startups.csv` | Hand-collected post-mortem summaries (pipe-separated). Verify against original sources before quoting; add your own rows |
| `seed.py` | Demo data |

## Database tables
`users`, `user_skills`, `projects`, `needs`, `members` (status: invited / joined), `messages`.

## Matching rule (the part to explain)
For each person, for each skill the project still needs: no skill = 0, has it = 0.6, plus up to 0.4 more depending on level vs the level required. Average over open needs, then `score = 85% skill fit + 15% availability (hours/10, capped at 1)`. Skills already covered by the team are ignored, so the list finds *complementary* people.
