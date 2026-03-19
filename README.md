### Hi there 👋

<!--
**jasongriller/jasongriller** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->


You are building a new feature for an existing React + TypeScript + Vite application called "Decomposer AI." This app is an internal AI-powered platform for software development teams, themed with a US Army design system. The new feature is an "AI Team Problem Intake" system — a structured request form and management dashboard that allows other teams across the organization to submit requests for the AI team to build something for them.

=== TECH STACK ===
- React 18 + TypeScript + Vite
- Tailwind CSS + custom CSS with CSS custom properties
- react-router-dom (createBrowserRouter)
- Lucide React icons (ONLY use lucide-react for icons)
- react-hot-toast for notifications
- Cognito auth via AuthContext (useAuth gives: userId, userEmail, userRole, isAuthenticated)
- API layer in src/services/api.ts using fetch + JWT auth headers (pattern: getAuthHeaders() → fetch with Bearer token)
- Role system: Admin, Pro, Basic (from src/types/roles.ts) — the intake form should be accessible to ALL authenticated users (any role can submit); the admin management/triage view should be behind RoleGuard with "canAccessAdminDashboard"

=== DESIGN SYSTEM (CRITICAL — match the app's existing light-mode style, NOT the exec dashboard dark theme) ===
Color tokens (CSS custom properties used throughout):
  --armygold: #ffcc01 (primary accent)
  --armyblack: #221f20 (primary text, navbar bg, dark surfaces)
  --armygreen: #2f372f
  --armyaccentgrey01: #565557 (muted text)
  --armyaccentgrey02: #d5d5d7 (borders, subtle bg)
  --page-bg: #e5e5e5 (page background)
  --card-bg: #fff (card background)
  --card-text: #221f20
  --card-muted: #565557
  --card-border: #d5d5d7
  --card-inset: #f2f2f2 (inset/nested areas)
  --core-darkblue: #00558c, --core-darkgreen: #007326, --core-red: #c00, --core-purple: #6b21a8

Dark mode is supported via html.dark class which overrides all tokens (e.g., --card-bg becomes #262b31, --card-text becomes #f2f3f5, --page-bg becomes #3b4047). ALL styles MUST use var(--token) instead of hardcoded colors so dark mode works automatically.

Component patterns to follow:
- Page wrapper: <Navbar /> at top, content in <main> with padding-top for fixed navbar, <MobileBottomNav /> conditionally via useMediaQuery('(max-width: 767px)')
- Section cards: white bg cards with 4px colored border-top, 0.5rem border-radius, var(--card-border) border, padding 1.25-1.75rem. Each card has a header row with a colored icon container (0.5rem padding, rounded, colored bg) + uppercase bold section title
- Form fields: labels are 0.8125rem, font-weight 600, color var(--card-text), with 0.75rem hint text in var(--card-muted) below. Inputs have 2px solid var(--card-border), 0.5rem border-radius, var(--card-bg) bg, 0.875rem font-size
- Buttons: Primary = bg var(--armyblack), text white, hover gold accent. Gold variant = bg var(--armygold), text var(--armyblack). All 0.5rem border-radius, font-weight 600
- Toast notifications via react-hot-toast (toast.success, toast.error)

=== FEATURE REQUIREMENTS ===

Create TWO components and supporting infrastructure:

### 1. IntakeForm.tsx — The Submission Form (accessible to all authenticated users)
A multi-step or sectioned form (NOT a single overwhelming wall of fields) where any team member can submit a request. Sections:

**Step/Section 1 — Requester Info**
- Name (text, pre-fill from userEmail if possible)
- Team / Department (text input with recent-values autocomplete/suggestions if possible)
- Priority (select: Low, Medium, High, Critical — with colored indicator dots)
- Target Completion Date (date picker)
- Date Submitted auto-populated

**Step/Section 2 — Context & History**
- "What changed recently?" (textarea, with helpful placeholder: "Describe the trigger — a new regulation, process change, leadership directive, etc.")
- "Why did it change?" (textarea)
- "Who or what is affected?" (textarea, placeholder: "Which teams, processes, or systems are impacted?")

**Step/Section 3 — Current State**
- "What systems/tools are involved?" (tag-style multi-input or textarea)
- "What does the current workflow look like?" (textarea, large — this is the most important context field)
- "What data is available?" (textarea, placeholder: "Describe data sources, formats, access, and any data limitations")
- "Known constraints or limitations?" (textarea)

**Step/Section 4 — The Ask**
- "Problem statement" (textarea, large, with guidance text: "Be as specific as possible. What problem are you trying to solve?")
- "What is your ideal outcome?" (textarea, placeholder: "Describe what success looks like — what would the finished product do?")

**Step/Section 5 — Stakeholders & Communication**
- Primary point of contact (text, default to submitter)
- Other stakeholders (textarea or tag input for names/emails)
- Preferred communication channel (select: Email, Teams, Slack, In-person)
- Any additional notes (optional textarea)

UX requirements:
- Progress indicator showing which step/section the user is on (styled stepper bar with gold active state)
- Form validation with inline error messages (red border + text) — problem statement and requester info are required, everything else is optional but encouraged
- "Save as Draft" functionality using localStorage so users don't lose work
- Confirmation screen after submission with a summary of what was submitted and a tracking reference number
- Smooth transitions between sections (CSS transitions, not jarring re-renders)
- A small "?" tooltip next to complex fields explaining why we're asking

### 2. IntakeManagement.tsx — Admin Triage Dashboard (admin-only)
A management view where the AI team can see, filter, sort, and triage all submitted intake requests. This should:

- Show a card-based list of all submissions (not a table), each card showing: requester, team, priority (color-coded badge), date submitted, problem statement preview (truncated), status
- Status workflow: New → Under Review → In Progress → Completed → Closed (with a status dropdown on each card)
- Filter bar: filter by status, priority, date range, team
- Sort by: date submitted, priority, target date
- Click a card to expand/open a full detail view showing all submitted fields in a clean read-only layout
- Ability to add internal notes/comments to a submission (not visible to the requester)
- Quick stats at top: total submissions, by status breakdown, avg time to review
- Search across all fields

### 3. API Integration (src/services/intakeApi.ts)
Create a service file following the exact same pattern as api.ts:
- Use getAuthHeaders() for JWT auth (import from api.ts or duplicate the helper)
- API_BASE from the same env var pattern
- Functions: submitIntakeRequest, listIntakeRequests, getIntakeRequest, updateIntakeStatus, addIntakeNote, deleteIntakeRequest
- All return typed interfaces
- Define TypeScript interfaces for IntakeRequest, IntakeNote, IntakeStatus, etc. in src/types/intake.ts

### 4. Route Integration
Add routes to src/routes.tsx:
- /intake — the submission form (protected, any role)
- /admin/intake — the management dashboard (protected, admin RoleGuard)

### 5. Navigation Integration
Add an "Intake" link to the Navbar component and also add it to the UtilitiesMenu dropdown as an option.

=== FILES TO CREATE ===
- src/components/IntakeForm.tsx
- src/components/IntakeManagement.tsx
- src/services/intakeApi.ts
- src/types/intake.ts

=== FILES TO MODIFY ===
- src/routes.tsx (add new routes)
- src/components/Navbar.tsx (add Intake nav link)
- src/components/UtilitiesMenu.tsx (add Intake to utilities dropdown)
- src/index.css (add intake-specific CSS classes following exec-dash pattern naming: intake-form-*, intake-mgmt-*)

=== IMPORTANT STYLE RULES ===
1. Use the app's LIGHT theme style (white cards on light gray bg), NOT the exec dashboard's dark gradient style
2. ALL colors via CSS custom properties (var(--token)) for automatic dark mode support
3. Follow the Settings.tsx pattern for section cards (border-top accent, icon + title header)
4. Follow the FeedbackModal.tsx pattern for form field styling
5. Use inline styles OR CSS classes in index.css — the app uses a mix of both (inline for dynamic/one-off, classes for reusable)
6. Mobile responsive — use useMediaQuery('(max-width: 767px)') and adjust layouts
7. Keep each file self-contained (no new external dependencies)

=== WHAT MAKES THIS EXCELLENT ===
- The form should feel INVITING, not bureaucratic — use friendly microcopy, placeholder text that guides thinking, and a progress bar that makes it feel approachable
- Smart defaults: auto-fill date, pre-populate name from auth, remember team/department from last submission
- The admin view should feel like a kanban-lite — quick visual scan of what's new, what's in progress, what's blocked
- Add subtle polish: transition animations on section changes, priority color coding consistent throughout, empty states with helpful illustrations/text
