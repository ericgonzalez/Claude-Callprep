---
name: call-prep-route
description: Runs the full ten-stage sales call-prep route on a company and returns a formatted PDF brief. The rep only supplies a company name (optionally a contact and deal type); the skill researches the account, earnings-call pain points, the contact's role, discovery questions, a specific opener, the buying committee, objections, competitors, a five-objection battle card, and a follow-up plan. Use whenever the user says "prep me for a call with X", "research [company] for a sales call", "brief me on [company]", "build a battle card for [company]", "run the route on [company]", or gives just a company name in a sales context.
---

# Call-Prep Route

The rep gives you a company. You run all ten stages of the route, in order, and hand back a PDF.
The rep should never have to write a research prompt.

Files in this skill:
- `references/prompts.md` - the ten prompts, their stop-rule fallbacks, and what "done" looks like. READ FIRST.
- `references/seller-profile.md` - a blank TEMPLATE of the rep's own product, proof points, pricing, method.
- `scripts/build_brief_pdf.py` - turns a JSON payload into the PDF. The schema is in its header docstring.
  It sits next to this SKILL.md. If you do not know its path, find it with
  `find / -name build_brief_pdf.py -path '*call-prep-route*' 2>/dev/null | head -1`.
- `assets/sample_payload.json` - a complete fictional payload showing the exact shape expected.

## 1. Intake (ask at most once)

Required: a company name. Everything else is optional.

- Contact (name and title), deal type, and stage are used if given. If not given, proceed with the
  likely buyer role and mark contact-specific content "inferred".
- Look for the rep's saved profile: a file named `call-prep-seller-profile.md` in the user's
  working folder (the workspace folder in Cowork, the outputs folder elsewhere). If it exists, read
  it. If not, read the blank template at `references/seller-profile.md` (the installed plugin is
  read-only, so never write to it), ask ONE batched question for the fields marked (*) (company,
  solution name, one-line description), and offer to save the answers as
  `call-prep-seller-profile.md` in the working folder so later runs skip the question. If the rep
  says to just run it, proceed and tag solution-fit content "inferred".
- Resolve the company yourself with a quick search. If the name is ambiguous (several plausible
  companies), ask one short question listing the candidates. Otherwise state the identity you
  chose in your first line ("Researching Harborline Logistics, harborline.com, 3PL") and continue.
  Wrong-company research is the worst failure, so verify the domain from search results rather than
  guessing it from the name.

Do not ask anything else before researching.

## 2. Run the route

Follow `references/prompts.md` stage by stage (01 to 10). Each stage feeds the next, so carry
findings forward: discovery questions must trace to stages 01-03, the opener to a dated finding,
the battle card to stages 06-08, the follow-up to everything.

Research budget: roughly 12-20 searches or fetches in total. Search each distinct item
separately (company news, earnings call, contact, competitors) rather than one combined query.
Prefer primary sources: company site, press releases, investor relations, SEC filings, the
contact's own public talks or posts. Use web_fetch on transcripts and press releases rather than
relying on snippets. Today's date matters: favour the last six months and record dates.

### The stop rule (non-negotiable)
Every stage ends with a fallback: if a source is gated or missing, say so and ask the rep to
paste it. Concretely:
- Never bypass a login wall or paywall (LinkedIn especially). Record it as a gap.
- Never invent names, quotes, prices, metrics, or customer results. Write "Not public" or
  "Needs rep input: <what>" instead.
- Every gap goes in `gaps[]` with a concrete ask ("Paste her last 3 posts").
- Tag each claim `sourced` (with source id) or `inferred`. Do not stack inference on inference.

### Quotes
Stage 02 needs exact wording. Use short verbatim excerpts (at most two sentences), attributed to
speaker and call, and link the source. Never paste long passages. If you cannot obtain exact
wording, drop the quote and tag the item "inferred".

## 3. Build the PDF

1. Write the payload to `/home/claude/payload.json` following `assets/sample_payload.json`. Keys are
   `meta`, `snapshot`, `s1_account_brief` ... `s10_followup`, `gaps`, `sources`. Set
   `meta.prepared_on` to today's date and `meta.sample` to false.
2. Write `snapshot.takeaways` LAST (3-5 sentences that trace to the stages).
3. Validate: `python <path-to>/scripts/build_brief_pdf.py /home/claude/payload.json --validate`.
   Fix every WARN that is a real problem (unknown source ids, wrong item counts). Empty stages are
   acceptable only if a matching entry exists in `gaps[]`.
4. Build: `python <path-to>/scripts/build_brief_pdf.py /home/claude/payload.json <outputs-folder>/<company>-call-prep-<YYYY-MM-DD>.pdf`
   where `<outputs-folder>` is `/mnt/user-data/outputs` in claude.ai, or the user's workspace
   folder in Cowork. If reportlab is missing: `pip install reportlab` (add
   `--break-system-packages` if the environment requires it).
5. Rasterize one or two pages (`pdftoppm -r 60 -png`) and look at them for clipped text or
   orphaned headings before delivering.
6. Share the PDF with the user (`present_files` where available; otherwise state the file path).

## 4. Reply in chat

Keep it short. Lead with the three findings most likely to change the call, then list the gaps that
need the rep's input, e.g. "Paste Dana's recent posts and I will sharpen the opener." Remind the
rep that the follow-up email is a draft: you never send anything. Do not restate the whole PDF.

## Rules that keep the brief trustworthy

- One source, one claim. A hire is a fact; "they are building an AI team" is an inference and is labelled.
- Proof points come only from the seller profile or sourced material.
- Competitor pricing is quoted only when public and cited; otherwise "Not public".
- Respect access boundaries: no scraping of gated pages, no workarounds.
- If the rep re-runs a company they have prepped before and gives prior notes, add what changed
  since last time at the top of the snapshot.
