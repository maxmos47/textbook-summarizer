# EM Gamification Platform — Agent Team Manifest

## Project Overview
**Product:** Emergency Medicine Resident Study & Gamification Web App  
**Main file:** `Web/index.html`  
**Backups:** `Web/Version/` (saved on explicit user command)  
**Current version:** v1.6 (6 backup versions archived)

---

## Team Roles

### Content Creator (CC)
**Persona:** Medical Education Specialist & Copywriter  
**Responsibilities:**
- Receive the user's raw idea and expand it into a full feature brief
- Define learning objectives, engagement hooks, and gamification logic
- Write all in-app copy: titles, tooltips, feedback messages, explanations
- Ensure clinical accuracy of EM content (aligned with attending physician persona)
- Output: Feature Brief handed to UI/UX Designer

**Triggers:** Any new idea or feature request from the user

---

### UI/UX Designer (UX)
**Persona:** Mobile-first Product Designer  
**Responsibilities:**
- Translate the Feature Brief into a concrete UI/UX specification
- Define screen layout, component hierarchy, color usage, and interaction flow
- Specify animations, transitions, and gamification visual feedback (XP bars, badges, etc.)
- Ensure consistency with existing design system (DM Serif Display / DM Sans, red #C0392B brand)
- Output: Design Spec handed to Programmer

**Design System:**
- Colors: `--red:#C0392B`, `--dark:#1a1a2e`, `--dark3:#0f3460`
- Fonts: DM Serif Display (headings), DM Sans (body)
- Radius: 12px / 18px / 24px
- Mobile-first, bottom navigation, sticky top bar

---

### Frontend/Backend Programmer (DEV)
**Persona:** Senior Full-Stack Developer (vanilla JS, HTML5, CSS3)  
**Responsibilities:**
- Implement the Design Spec inside `index.html` (single-file app)
- Write clean, well-commented JavaScript
- Handle all data persistence (Firestore), state management, and routing between screens
- Integrate gamification mechanics: XP, levels, streaks, badges, leaderboards
- Output: Updated index.html handed to Bug Finder

**Constraints:**
- Single file (`index.html`) — no external JS/CSS files
- CDN-OK for fonts and icons only
- Firebase Firestore for data persistence (custom SHA-256 auth, no Firebase Auth)
- Firebase Storage for image uploads (uploadString with data_url)
- ES module script (`<script type="module">`) — must run via HTTP server, not file://

---

### Bug Finder & Debugger (BUG)
**Persona:** QA Engineer & Code Reviewer  
**Responsibilities:**
- Review the programmer's output for logic errors, UI glitches, and broken interactions
- Check mobile responsiveness and cross-browser compatibility
- Verify gamification math (XP calculations, level thresholds, badge triggers)
- Ensure no regressions against previously working features
- Fix bugs directly in the code
- Syntax check via Node.js `node --check` after every major edit
- Output: Final verified index.html + Bug Report handed to Team Lead

---

## Workflow Pipeline

```
START OF NEW VERSION
      |
      v
/superpowers:using-superpowers   ← MANDATORY FIRST STEP
   -> Invokes skill discovery, sets workflow discipline for the session
      |
      v
Content Creator
   -> Expands idea, defines scope, writes copy & clinical content
      |
      v
UI/UX Designer
   -> Designs screens, components, interactions, gamification visuals
      |
      v
Programmer
   -> Implements in index.html
      |
      v
Bug Finder
   -> Tests, fixes, validates (node --check syntax)
      |
      v
Team Lead -> User
   -> Summary of changes + backup saved
```

---

## Backup Protocol
- User commands: "Save version" or "Backup" -> copy index.html to Version/index vX.X.html
- Current versions on disk: v1.0, v1.1, v1.2, v1.4, v1.5, v1.6
- Next backup will be: **v1.7**
- NEVER use Edit tool for large blocks — use Python string replacement to avoid emoji truncation

---

## Technical Notes (Critical for DEV)
- File edits: Always use Python `str.replace()` via bash, NOT the Edit tool for large JS blocks (emoji chars cause truncation)
- Syntax check: `node --check /tmp/check.js` after every JS change (strip CDN imports first)
- Firebase: `collection`, `getDocs`, `getDoc`, `setDoc`, `updateDoc`, `deleteDoc`, `writeBatch`, `doc` all imported
- Firebase Storage: `ref`, `uploadString`, `getDownloadURL`, `deleteObject` imported
- NO `addDoc` imported — use `setDoc(doc(collection(db,'col')), data)` for auto-ID docs
- Session persistence: `localStorage` key `em_hs_session` stores username
- Avatar render: `_avatarHtml(avId, bdId, size)` — central function for all avatar display
- Host check: always use `currentUser.role === 'host'` (NOT `currentUser.isHost`)
- Cooldown helper: `fmtCd(ms)` formats milliseconds as "Xh Ym"
- Config doc: `config/settings` in Firestore stores all host-configurable settings

---

## Current App Features (as of v1.6)

### Auth & Session
- Login / Register screen with SHA-256 password hashing
- Persistent login session via localStorage (`em_hs_session`)
- Invite code gating for registration

### App Shell
- Bottom navigation (Library, Ranks, SAQ, Profile, Shop, Host)
- Host-only nav (Host panel, no resident tabs)
- Dark sticky top bar with XP badge + coin display
- XP bar with level system

### Study System
- Chapter-based study with icon picker (host editable)
- Chapter quiz with cooldown (host configurable)
- Chapter leaderboard sub-tab in Ranks

### Random Pool Quiz
- Timed MCQ from a host-managed question pool
- Cooldown between attempts (host configurable)
- Pool Quiz Challenger leaderboard sub-tab in Ranks

### Gamification
- XP + coins for correct answers
- Level system with XP bar
- Badges (achievement system, host managed)
- Titles (cosmetic, host managed)
- Daily login reward

### Shop & Profile
- Shop: buy avatars, borders, titles with coins
- Animated avatar borders (tier-based: static -> shimmer -> glow pulse -> elite aura -> rainbow spin -> fire flicker)
- Animated emoji bob for premium avatars
- Instant buy feedback (optimistic UI, rollback on error)
- Profile "Customize" tab: equip owned avatars, borders, titles

### Leaderboard (Ranks tab)
- 4 sub-tabs: Total XP | Random Quiz | Chapter | Image Quiz (Photos)
- Image Quiz leaderboard sorted by imageQuizScore

### Image Quiz & Time Attack
- Host creates image quizzes: upload image to Firebase Storage, MCQ, explanation, XP reward
- Image tab: Time Attack landing page only (no browsable grid)
  - Shows user's Image XP, total cases, round config pills, cooldown status
  - Live countdown timer when on cooldown
- Time Attack mode: timed MCQ session
  - Configurable: questions per session (5/10/15/20), seconds per question (10-45)
  - XP and coins per correct answer: manual input 0-100 (host sets)
  - Cooldown between sessions: 0-24h (host sets)
  - Per-question countdown ring (SVG), auto-advance 3s after answer
  - Times-up auto-reveals correct answer
  - End screen: score summary, XP earned, coins earned, Play Again
  - XP feeds both `imageQuizScore` (Image LB) and `score` (Total XP LB)
  - `lastTimeAttack` timestamp saved to Firestore on session complete

### Host Panel
- Chapter Manager, Category Manager, Badge Manager, Title Manager
- Quiz Pool Manager (upload questions, manage pool)
- Pool Quiz Settings: question count, time per Q, cooldowns, pass cut-off
- Image Quiz Manager: create/delete image quizzes
- Time Attack Settings: questions, seconds/Q, XP (0-100), coins (0-100), cooldown
- Leaderboard Control, Member Manager
- Invite code management

---

*Last updated: 2026-04-29*
