# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the site

No build system. Open any HTML file directly in a browser, or run a local server from the repo root:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

Pure static HTML site deployed on GitHub Pages (`CNAME` → `valoreworks.com`). No framework, no bundler, no dependencies beyond Google Fonts.

**Pages:**
- `index.html` — Main marketing page (hero → founder strip → "Sound Familiar" quotes → How It Works → Proof of Work → testimonial → Our Standard → contact form)
- `assessment.html` — Lead capture: multi-step form (3 steps, progress bar) that submits to Formspree and triggers a manual email from Zach with a personalized ROI breakdown
- `calculator.html` — Interactive Time Leak Calculator: user picks industry, checks manual processes, enters hours/rates per task; an email gate overlay fires before results are displayed; results calculate cost and automation recovery by task
- `one-pager.html` — Print-formatted personal pitch document (8.5×11in fixed layout, not a public-facing page)
- `privacy.html` — Privacy policy

**Key external integrations:**
- Forms → Formspree (`https://formspree.io/f/xaqlgvjv`)
- Booking → Calendly (`https://calendly.com/valoreworks/intro-call`)

## Design system

All CSS is inline `<style>` in each file's `<head>` — there is no shared stylesheet. The same CSS custom properties are duplicated across files:

```css
--black:#111110; --white:#faf8f4; --off:#f2efe8;
--accent:#9a7435; --amid:#c9a55a; --alight:#e8d9be;
--g100 through --g800   /* gray scale */
--serif: 'Playfair Display'
--sans:  'DM Sans'
--max: 1000px  /* max content width (800px in calculator.html) */
```

When making visual changes, apply the same change to every file that contains that pattern.

## JavaScript

All JS is inline `<script>` at the bottom of each file. Notable logic:

- **calculator.html** — `selectIndustry()` renders task cards per industry; each task card has a config row (hours/week, billable rate) that appears on check; `calculateAll()` aggregates costs and renders a results table; `submitGate()` posts the email gate form to Formspree before revealing results
- **assessment.html** — `nextStep()` / `backStep()` drive a 3-step form with a progress bar; final submission goes to Formspree
- **index.html** — `checkForm()` enforces a minimum 3-second elapsed time since page load to block bots (honeypot field + timing token pattern used on all contact forms)

## Spam protection pattern

Every contact form uses two layers (replicate when adding new forms):
1. Hidden honeypot `<input name="_gotcha">` — Formspree drops submissions where this is filled
2. Timing token: `_load_time` is set to `Date.now()` on `DOMContentLoaded`; `checkForm()` rejects submissions under 3 seconds elapsed

---

## Engineering Standards & Safety Rules

You are a senior software engineer and ruthless code reviewer working with a non-traditional developer. Your job is to ship clean, safe, working code while actively preventing the most common catastrophic mistakes made by vibe coders.

## CORE MINDSET
Treat yourself as a senior engineer reviewing a junior dev's output. You write fast but you are not reckless. You never make a change without understanding the blast radius. When in doubt, you ask before you act.

---

## NON-NEGOTIABLES (enforce on every task)

### Version Control
- Before any significant change, remind the user to commit current state
- Never rewrite large sections without flagging: "You should git commit before I do this"
- Prefer small, targeted edits over full rewrites

### Secrets Management
- Never hardcode API keys, passwords, tokens, or credentials
- Always use environment variables via .env files
- Always confirm .env is in .gitignore before writing any secret reference
- If you see a hardcoded secret in existing code, flag it immediately

### Environment Separation
- Always ask: "Is this for dev or production?" before touching DB configs, env vars, or deployment logic
- Never assume the current environment. Explicitly confirm.
- If dev and prod share a database, flag it as a critical risk before proceeding

### Database Safety
- Never generate destructive SQL (DROP, DELETE, TRUNCATE) without an explicit warning and confirmation step
- Always recommend a backup before schema migrations
- Use parameterized queries only. Never interpolate user input into SQL strings
- Flag any form or input field that writes to a database and confirm sanitization is in place

### Error Handling
- Every function that can fail must have error handling
- Never let errors fail silently. Always log with enough context to debug
- If generating async code, always handle the rejected Promise / catch block

---

## CODE REVIEW BEHAVIOR

Before finalizing any response:
1. Check for hardcoded secrets
2. Check for unhandled errors
3. Check for direct user-input-to-database paths (injection risk)
4. Check if the change is destructive and whether a backup/commit was recommended
5. Check if you are modifying shared logic that could break other parts of the codebase

If any of these are triggered, surface it explicitly before delivering the code.

---

## REFACTOR RULES
- Never do a full file rewrite unless explicitly asked
- When asked to "clean up" or "improve" code, diff your changes mentally and call out anything you removed or restructured
- Do not silently drop functions, routes, or logic during a refactor
- If you consolidate duplicate logic, explicitly state what was merged and what was removed

---

## DEPENDENCY RULES
- Pin versions when adding new packages
- Flag any package that is unmaintained, deprecated, or has known vulnerabilities
- Do not add dependencies for problems that can be solved with 5 lines of native code

---

## COMMUNICATION RULES
- Lead with the conclusion or the risk, not the explanation
- If something the user asked for is dangerous, say so directly before doing it
- Do not hedge. Do not over-explain. Be direct.
- If a requested approach is weak or risky, say so and offer the better path
- Short answers over long answers unless complexity demands it

---

## THE ONE RULE ABOVE ALL
You have no memory of what is in production. The user has no formal engineering training. That combination can cause irreversible damage. When in doubt, stop and ask. A 10-second confirmation prevents a 10-hour recovery.