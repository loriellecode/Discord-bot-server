# Server Rebuild + Bot Plan

Status: planning only. Not connected to the server yet. Nothing here has been run.
Scope change: the server drops peer support and becomes a community/hangout server with staff, events and safety tooling.

## 0. Open items to settle before building

1. **Old bot code.** The repo was empty when this plan was written. To match how the old bot did things (command names, embed style, role names, channel IDs), add it to this repo or tell me where it lives.
2. **1-on-1 with Bubble.** Two messages conflict: "update (legally okay)" and "remove". This plan follows the latest one (**remove it**). The legal wording is dropped.
3. **Minors + sexual content.** The server has under-18 members and you want an 18+ channel for sexual or trigger content. Discord's rules require sexual content to be in age-restricted channels and forbid it around minors. So:
   - The 18+ channel is hidden until a member is verified 18+ (see 4.7).
   - Under-18 members never see it. The bot must not be able to grant the role without verification.
   - My recommendation: rename it **18+ Vent / Trigger Warning**, and say sexual content is only allowed there. That matches your request.
4. **Hotline accuracy.** Numbers in the Emergency channel must be re-checked against official sources on the day we post them. Numbers change.
5. **Legal / safety.** Moving off peer support is the right call. The Emergency channel must clearly say the server is not a crisis service.

## 1. Final channel layout

```
WELCOME
  #welcome          (introduction merged in; what the server is, what channels exist)
  #rules
  #roles-and-apply  (one channel: all role applications + advisor info)
  #suggestions

ANNOUNCEMENTS
  #announcements    (server news, updates, events, new features)

EMERGENCY
  #emergency        (hotlines worldwide, "not a crisis service", when to contact an adult)

RESOURCES
  #resources        (affordable therapy guide, other resource documents)

COMMUNITY
  #general
  #18-plus-vent     (age-restricted, verified 18+ only, sexual/trigger content allowed only here)
  #bot-commands     (optional, keeps bot spam out of general)

VOICE
  General Voice     (ping role/message when someone joins)
  <2-3 static rooms>   (no auto-created duplicates)
  Event Voice / Event Stage: see 4.5 (temporary, created only for events)

STAFF (hidden)
  #staff-announcements
  #staff-docs       (policy + moderation PDFs, sign-off)
  #staff-mod-log    (human mod actions)
  #bot-mod-log      (automatic bot actions)
  #bot-controls     (admin-only bot config/commands)
```

Welcome category has 4 channels (you asked for 3-4): welcome, rules, roles-and-apply, suggestions.

### Channels to delete
- Human circle network
- 1-on-1 with Bubble
- Info desk (its useful content, "what's in the server", moves into #welcome)
- Event news (events go to #announcements)
- Separate advisor info channel (merged into #roles-and-apply)
- All peer-support channels
- Any duplicate or auto-generated voice channels
- Separate apply-for-roles channels (merged into one)

### Channels to rename / move
- Old "resources" -> **#emergency** (emergency info only). Everything else from it -> new **#resources**.
- Introduction -> merged into #welcome.
- Suggestions -> Welcome category.
- Events and new features -> #announcements.

## 2. Channel content

### #welcome
Plain-language blurb, a map of the channels, and a note that the server is not a crisis service. Replaces Info desk and Introduction.

### #rules (1-2 sentences each, no repeated info)
1. Be respectful. No harassment, hate or bullying.
2. Do not hurt or harass minors.
3. Listen to staff.
4. Sexual or trigger content is only allowed in the 18+ channel.
5. Under 18? You can contact an adult at any time if someone makes you uncomfortable.
6. Going to DMs is at your own risk. Staff cannot moderate DMs.
7. Punishment: **warning 1, warning 2, then a ban.** Each warning comes with a 30 minute timeout. If the circumstances are unclear, there is due process.
8. Contact: you can DM a moderator about a warning and you will be given space to argue it.

Removed from the old rules: the section on how staff deal with moderation (moves to the staff docs).
Kept: the "unclear circumstances -> due process" line.

### #emergency
- "This server is not a crisis service."
- If you are in immediate danger, contact local emergency services.
- Worldwide hotline list grouped by region, plus the international directory (findahelpline.com). Verify every number before posting.
- Minors: contact a trusted adult.

### #resources
- Document: **How to find affordable therapy**. Cover sliding-scale clinics, community mental health centers, university training clinics, nonprofit and employer (EAP) options, and low-cost online options, with a regional-agnostic structure and links to directories. Draft it as a doc and have you review it.
- Other resource links moved from the old channel.

### #roles-and-apply
One channel. A pinned embed with button/select menu per role. Each role lists requirements. The advisor info is condensed into a short section. Applications open a form (modal) and post to a private staff review thread.
**Head Moderator is not listed.** It is voted on by current staff (see 4.4).

## 3. Roles

| Role | How obtained |
|---|---|
| Member | Auto on join after accepting rules |
| 18+ Verified | Manual verification by a mod only |
| Moderator | Application -> staff review |
| Advisor | Application (info in #roles-and-apply) |
| Head Moderator | Voted by current staff, not applied for |
| Admin / Owner | You |

Mod hiring: with the application flow live, you post the call in #announcements. The bot cannot hire anyone. You and staff decide.

## 4. Bot features

### 4.1 Tech stack (assumed, confirm against the old bot)
discord.js v14 (Node). SQLite or Postgres for warnings, votes, sign-offs. Config in a single `config.json` or env file (token, guild ID, channel/role IDs). Use slash commands.

### 4.2 Server setup command
`/setup` (admin-only): creates the layout in section 1 idempotently. It reads or creates by name and does not duplicate. A `--dry-run` flag prints what it would change. Deletion of old channels happens as a separate `/cleanup` step that lists channels first and requires a confirm button. Nothing is deleted without confirmation.

### 4.3 Warning system
- `/warn @user reason` (mods only).
- The bot DMs the user: what rule was broken, the reason, which warning number they are on (1 or 2), what happens next, and how to contact a moderator and argue it.
- Applies a **30 minute timeout** (Discord timeout).
- Warning 1 and 2 are warnings. The third strike is a ban. Escalation is automatic as a suggestion to staff; a human confirms the ban.
- Logs every action to #bot-mod-log with moderator, user, reason and warning count.
- If the DM fails (DMs closed), post a note in #staff-mod-log.
- `/warnings @user`, `/clearwarn @user` (head mod+) for reviewing and reversing.
- Appeal: a button in the DM opens a private thread with staff.

### 4.4 Head moderator vote
`/headmod-vote start` opens a poll visible to current staff only. One vote per staff member, nominations from staff only. The bot tallies and posts the result in #staff-announcements. Admin confirms the role grant.

### 4.5 Events
- Staff run `/event create` (name, time, description, needs voice or stage?).
- The bot posts to #announcements.
- If the event needs voice or stage, the bot creates a temporary channel shortly before the start and deletes it after. This replaces permanent Event Voice and Event Stage channels. (If you want permanent ones instead, they go in the Voice category and the temporary part is skipped.)

### 4.6 Voice
- No join-to-create channels, so no duplicates.
- Static voice channels only.
- **General Voice ping:** when someone joins an empty General Voice, the bot posts a message in #general mentioning a "Voice Pings" opt-in role. Cooldown (default 15 min) to avoid spam.

### 4.7 18+ verification
Mod-run `/verify @user` after manual checking. Grants the 18+ role. Logged to #bot-mod-log. No self-serve button.

### 4.8 Staff documents + sign-off
- Two PDFs, **Staff Policy** and **Moderation Guide**, posted in #staff-docs as attachments.
- Each has an "I have read and agree" button. Clicking records the user ID, document version and timestamp in the DB.
- `/staff-signoffs` shows who has and has not signed. The bot reminds unsigned staff. New staff cannot be granted the mod role until signed (soft check with a warning).
- Version bump re-requires signatures.
- The PDFs hold the details removed from the public rules (how staff handle moderation).

### 4.9 Logs
- **#staff-mod-log:** manual actions by humans (notes, bans, kicks).
- **#bot-mod-log:** automatic bot actions (warnings issued, timeouts, joins, verification).

### 4.10 Bot controls
`#bot-controls`, admin-only. Slash commands for: toggling the voice ping, setting its cooldown, opening or closing applications, editing the rules embed, forcing re-post of embeds, and viewing config.

### 4.11 "Fix bot development and updates"
Needs the old bot's code to scope. Planned baseline:
- Git branches (`main` stable, feature branches).
- Version number in the bot and a `/version` command.
- Release notes auto-posted to #announcements when an update ships.
- Basic error logging to a private channel or file.
- A README with setup steps.

## 5. Build order

1. Get the old bot code, decide stack, scaffold repo (README, config, .env.example).
2. `/setup` + `/cleanup` (dry-run first).
3. Rules, welcome, emergency and resources content posted by the bot.
4. Roles, applications, 18+ verification.
5. Warning system and logs.
6. Voice changes and ping.
7. Events.
8. Staff docs, sign-off, head mod vote.
9. Bot controls and update/release flow.
10. Test in a throwaway server first, then run against the real one.

## 6. What I need when we connect

- The old bot's code (or a description of its commands and style).
- Bot token and guild ID (give them to the host's secret store, not into chat or the repo).
- The list of current channels and roles (a screenshot or export is enough).
- Who current staff are (for the vote and sign-off).
- Which hotlines or regions matter most to your members.
