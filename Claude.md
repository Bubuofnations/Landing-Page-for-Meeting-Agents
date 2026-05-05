
# AI Meeting Agent — Project Instructions

## Project Overview

This is the **AI Meeting Agent** project: a proactive AI meeting assistant (POC → Full Autonomous Agent) that listens to live meetings, surfaces real-time insights to the user, and ultimately attends meetings autonomously on the user's behalf.

**POC Goal:** Prove the core loop — Capture → Understand → Reason → Proactively Surface — with a small cohort (5–10 beta users) before investing in scale.

**North Star:** Full Autonomous Meeting Agent that attends, speaks, negotiates, and concludes meetings on the user's behalf, including agent-to-agent meetings.

---

## Getting Started on a New Machine

1. **Clone the repo:**
   ```bash
   git clone https://github.com/bouboubusayo/Meeting-agents.git
   cd Meeting-agents
   ```

2. **Read in this order:**
   - [`claude.md`](claude.md) (you are here) — quick reference
   - [`Resources/Design_Spec.md`](Resources/Design_Spec.md) — all 9 screens, components, UX copy, accessibility
   - [`Resources/Technical_Architecture.md`](Resources/Technical_Architecture.md) — detailed tech design
   - [`Resources/UX_Research_Report.md`](Resources/UX_Research_Report.md) — user research, personas
   - [`Resources/Product_Strategy_Report.md`](Resources/Product_Strategy_Report.md) — market, GTM, pricing
   - [`Resources/POC_PRD.md`](Resources/POC_PRD.md) — feature scope, acceptance criteria

3. **Start development:**
   ```bash
   cd web
   npm install
   npm run dev
   ```
   App runs at `http://localhost:3000`

---

## Project File Structure

```
/Users/busayoodesanya/Meeting Agents/
├── claude.md                             ← Project quick reference
├── Resources/
│   ├── Design_Spec.md                    ← 9 screens, components, design system
│   ├── Technical_Architecture.md         ← Audio, STT, LLM, data flow
│   ├── UX_Research_Report.md             ← User research, personas (5x)
│   ├── Product_Strategy_Report.md        ← Market sizing, GTM, pricing
│   └── POC_PRD.md                        ← Feature spec, acceptance criteria
├── web/                                  ← Next.js + Tailwind + shadcn/ui frontend
│   ├── app/                              ← App Router pages
│   ├── components/                       ← shadcn/ui + custom components
│   ├── lib/                              ← Utilities
│   └── package.json
├── skills by me/                         ← Project-specific skill files
│   ├── engineering-manager.md
│   ├── senior-engineer.md
│   ├── senior-frontend-developer.md
│   ├── senior-designer.md
│   ├── product-manager.md
│   ├── ux-researcher.md
│   └── ux-research.md
└── skills/                               ← Global Claude Code skills
```

---

## Available Skills

These skills are available via slash commands. When the user says "engineer", "engineering manager", "senior engineer", "front-end developer", "UX researcher", or "product manager", invoke the corresponding skill.

| Skill | Command | When to Use |
|---|---|---|
| Senior Software Engineer | `/senior-engineer` | Architecture, technical tradeoffs, cost estimation, data flow |
| Engineering Manager | `/engineering-manager` | Budget decisions, simplicity reviews, team coordination, POC scope |
| Senior Frontend Developer | `/senior-frontend-developer` | UI components, Chrome extension, real-time UI, design fidelity |
| Senior Designer | `/senior-designer` | Screen design, design system, UX patterns, visual hierarchy |
| UX Researcher | `/ux-researcher` | User research, persona validation, trust analysis, research plans |
| Product Manager | `/product-manager` | Scope, prioritisation, GTM, pricing, investor narrative |

**Note:** Skills can and should collaborate. When a question spans multiple disciplines, invoke multiple skills and synthesise their outputs.

---

## Security Requirements (Non-Negotiable)

Before writing or reviewing any code, always consult these three security skills:

1. **`/security-review`** — General security review for all code
2. **`/security-scan`** — Automated security scan
3. **`cloud-infrastructure-security`** — Cloud and infrastructure security (referenced in `/security-review`)

Security is a constraint, not a feature. No code ships without passing security review.

---

## Key Architectural Decisions (Locked)

### Audio Capture
- **Primary:** Chrome extension (`chrome.tabCapture`, Manifest v3) — one-time install, automatic per session
- **Fallback:** `getDisplayMedia` + `getUserMedia` dual-stream merge — for users who have not installed the extension
- **NOT supported:** Zoom desktop app, Teams desktop app, mobile
- **Supported platforms:** Google Meet, Zoom web, Teams web — Chrome/Chromium desktop only

### Speech-to-Text
- **Deepgram Nova-2** — streaming, `diarize: true`, `diarize_version: "3"`
- No audio stored — transcript-only storage, enforced at backend boundary

### LLM Reasoning
- **Claude Haiku 4.5** — streaming, structured JSON output, signal detection
- User identity from Google auth (`user.user_metadata.full_name`) injected into prompts

### Speaker Identification (POC)
- Deepgram diarization gives generic Speaker 0, 1, 2...
- LLM inference from attendee list + conversational context (~60–70% accuracy)
- Post-session `<SpeakerMappingPrompt>` — user confirms mappings, stored in Supabase for future meetings
- v1: DOM scraping of Google Meet / Zoom web for real-time names

### Authentication
- Supabase Auth + Google OAuth 2.0
- Must include `access_type=offline&prompt=consent` to receive `refresh_token`
- Email + password as fallback
- No enterprise SSO for POC

### Database
- Supabase (Postgres + pgvector)
- `user_integrations` table for encrypted refresh tokens

### Frontend Stack
- Next.js 14+ (App Router) + Tailwind CSS + shadcn/ui
- Chrome Manifest v3 extension
- Deployed to Vercel (free tier)

### Backend
- Node.js WebSocket server on Railway ($5/month after trial)

---

## Signal Types

| Signal | Colour | Trigger |
|---|---|---|
| `decision` | Blue `#2563EB` | Decision forming or being made |
| `alignment_gap` | Orange `#F97316` | Disagreement or misalignment detected |
| `commitment` | Amber `#F5A623` | Commitment made by any participant |
| `risk` | Red `#DC2626` | Risk or blocker raised |
| `tone` | Purple `#7C3AED` | Significant tone shift |
| `direct_question` | Amber `#F5A623` | Someone directly addresses the user for their opinion |

`direct_question` confidence threshold: 0.8. User identity resolved from Google auth — no additional auth step required.

---

## POC Budget (Engineering Manager Governs This)

- **Total infrastructure budget:** $100–200 until first paying users
- Deepgram: $200 free credit (~565 hours of audio)
- Claude Haiku 4.5: ~$0.01/meeting
- Supabase + Vercel: free tier
- Railway: $5/month after trial
- Do not use Recall.ai — the Chrome extension eliminates that cost

---

## Design Principles (Non-Negotiable)

1. Under 3 minutes from sign-up to first value — no exceptions
2. Four onboarding steps: Welcome → Connect Calendar → Install Extension → Ready (~110 seconds total)
3. Proactive not reactive — AI surfaces insights without being prompted
4. Transparency by design — users always know what the AI can see and hear
5. Distraction-free — the overlay must not affect meeting platform performance
6. Honest about platform support — Chrome/Chromium desktop + browser-based meetings only; state this on every relevant screen

---

## Onboarding Flow (Final — Do Not Change Without EM + Designer Approval)

```
Step 1: Welcome           (~30s)
Step 2: Connect Calendar  (~60s)  — Google primary; Outlook "coming soon"
Step 3: Install Extension (~15s)  — one-time install; auto-advance on detection
Step 4: Ready             (~5s)
```

Total: ~110 seconds.

---

## Important Resource Links

**For UI/Component Implementation:**
- [`Resources/Design_Spec.md`](Resources/Design_Spec.md) — sections 3–7: screen specs, component definitions, UX copy, accessibility

**For Backend/Integration:**
- [`Resources/Technical_Architecture.md`](Resources/Technical_Architecture.md) — audio capture, STT, LLM pipeline, data flow

**For Product Decisions:**
- [`Resources/Product_Strategy_Report.md`](Resources/Product_Strategy_Report.md) — market, personas, pricing tiers
- [`Resources/UX_Research_Report.md`](Resources/UX_Research_Report.md) — user interviews, trust findings

**For Feature Scope:**
- [`Resources/POC_PRD.md`](Resources/POC_PRD.md) — MVP scope, acceptance criteria

---

## Working Conventions

- Do not address the user by name in responses
- Lead with the decision or action, not the reasoning
- When multiple skills collaborate, present a unified output
- Always read [`Resources/Design_Spec.md`](Resources/Design_Spec.md) before making design or implementation decisions
- All code must pass security review before committing
- Commit and push to GitHub after every meaningful change

---

**Last Updated:** April 2026
**Status:** POC — Design_Spec.md complete. Web app scaffolding ready. Next: build prototype screens.
