# UI/UX Specification

Related: [`PRD.md`](../PRD.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`02-architecture.md`](02-architecture.md)

Reference screenshots: `docs/reference/` (four sheets; evidence of arrangement, colors and labels, not of schema or formulas). Per-dashboard layouts: [`06-dashboard-catalog.md`](06-dashboard-catalog.md).

## 1. Design principles

- **Professional analytics first:** legible data, clear hierarchy, minimal decoration, predictable interactions.
- **Dense but not cramped:** keep the Tableau view's information density with better spacing.
- **Configurable, not arbitrary:** flexible layout inside a stable design system and supported widgets.
- **Consistent semantics:** same metric, status, filter and action look and behave the same everywhere.
- **Responsive by default:** adapt to available width/height; the screenshot resolution is not the layout spec.
- **Accessible by design:** keyboard, labels, contrast, chart alternatives, reduced motion.

## 2. Functional UI requirements

### Navigation and shell
- `UI-001` `[C]` Collapsible sidebar with accessible toggle and modular dashboard sections; new dashboards can be added one at a time without rewriting the shell.
- `UI-002` `[C]` Charts resize when the sidebar collapses, the browser resizes or orientation changes.
- `UI-003` `[P]` Each dashboard shows title, optional description and last-updated timestamp.

### Filters and state
- `UI-004` `[C]` Global, dashboard-level and widget-level filters with explicit scope; validated on client and server.
- `UI-005` `[C]` Cancel stale requests; out-of-order responses never overwrite newer results.
- `UI-006` `[P]` Shareable URL state where safe; reset-filters action.
- `UI-007` `[C]` Filter changes update dependent cards and charts without a full page reload; loading, empty, stale and error states exist.
- `UI-017` `[C]` Principal buttons (ASW, INA) in the top bar. Only allowed principals appear; a single allowed principal is preselected and shown as a label. Switching principal reloads data and resets filters whose values belong to the other principal.
- `UI-018` `[C]` Region, branch and similar filters list only the user's allowed values; a single allowed value is preselected. If a saved link or playlist points outside the user's scope, show a clear "not available for your access" state, not an empty chart.
- `UI-019` `[C]` Admin access screen: manage roles, capabilities, data scopes (`all` or a list per dimension) and user assignments; shows an effective-access preview for a chosen user; confirms destructive changes.
- `UI-020` `[C]` **AI Chat icon** in the right panel (visible only with `ai.chat.use`). Click opens the chat box. Visual answers open as a pop-up (chart, card, line, bar) that hides on close, background click or another menu click; each result stays in the chat as a clickable item that reopens the pop-up. Details and states: `07-ai-chat.md` section 7.
- `UI-021` `[C]` **Sign-in page:** username and password form (Microsoft sign-in deferred). "Change password" screen after login (forced at first login). Failed or locked sign-in shows a generic message; a disabled account shows "no access, contact your administrator".
- `UI-022` `[C]` **Admin user screen:** create user (name, username, temporary password), choose roles, show onboarding information to copy, reset password, enable/disable. **Admin access screen** switches each feature and filter on or off per role (`hidden`/`locked`/`editable`), with an effective-access preview.
- `UI-024` `[C]` Every dashboard and AI result shows "data as of <last completed data refresh>"; the current month is labelled month-to-date.
- `UI-025` `[C]` AI chat: when the AI asks to confirm a typo or an option, it shows buttons (Confirm / alternatives) and suggested questions; results appear only after confirmation.
- `UI-026` `[C]` **Settings > AI Provider** (`ai.settings.manage`): list of profiles with status, create/edit profile (type, base address, model, API key as a write-only field showing last 4 characters), Test connection, Test tool calling, Set active, data-sharing acknowledgement for non-local providers, gateway checklist (fallback off, compression off), global AI settings. Changes are confirmed and audited.
- `UI-027` `[C]` **Admin user screen: MFA button.** Each user has an Enable / Disable MFA button, an MFA status, and a Reset MFA action. Users with MFA on see the enrollment screen (QR or key, recovery codes shown once) at next sign-in and a code prompt after the password.
- `UI-028` `[C]` **Auto-refresh behavior:** on a data-refresh event the dashboard updates in place (no full reload or flicker), the as-of time changes, and a small non-blocking notice says "Data updated". During a running refresh an optional "refresh in progress" indicator is shown. Rotation and kiosk displays update the same way.
- `UI-029` `[P]` **Executive Summary and Regional Manager view** as laid out in `09-management-experience.md` section 4; phone layout is a single column of cards with large touch targets (`MGMT-012`).
- `UI-030` `[C]` **Sales Detail (pivot) controls:** a period range picker (start and end from the available periods), a field picker for dimension headers (choose and reorder) and for fact columns, the table with grand total (and optional subtotals with expand/collapse), sorting and filters, saved views, Export button. Wide tables scroll inside their own container with a frozen header and first column.
- `UI-033` `[C]` **Drill-down UI** (`PIVOT-014` to `PIVOT-020`): expand/collapse chevrons on rows, a breadcrumb and level badge, a Flat / Drill mode switch, an "expand to level" control. A level with no data is shown but its measures read "not available at this level" with a tooltip saying why; the expand control never leads to a blank screen. Keyboard-operable tree grid with announced level and state.
- `UI-034` `[C]` **Department switcher** in the top bar and a sidebar listing the selected department's dashboards; planned departments show a "Coming soon" page; disabled ones are hidden (`DEPT-009` to `DEPT-014`). The shell takes titles, icons and navigation from department data and never hard-codes Sales wording.
- `UI-035` `[C]` **Admin screens:** Departments (create, order, status) and, on the user screen, a switch per department plus that department's role(s) and an effective-access preview (`DEPT-015` to `DEPT-018`).
- `UI-036` `[P]` **Daily progress chart** (planned, `PROG-001`): cumulative achievement line against the working-day pace line, colored by the confirmed rule, with the tooltip date, target, real to date, % and pace %; table alternative for accessibility; shown only where a day-level source is registered.
- `UI-031` `[P]` **Info icon on every metric label** opens the glossary entry (definition, formula, source, as-of) (`MGMT-011`).
- `UI-032` `[P]` **Export and Share** buttons on Sales Detail and the Executive Summary; a share link to out-of-scope content shows the "not available for your access" page.
- `UI-043` `[C]` **Settings > Notifications:** email form with write-only password, "Send test email", status and last result, an on/off switch, and a note that nothing is sent until configured (`NOTIF-001` to `NOTIF-006`). A data-owner contact screen is available even before email is configured.
- `UI-023` `[P]` **Data Findings sheet** and small data-quality indicators on affected elements (`08-data-findings.md`); manual-upload figures carry a source label and latest-month note.
- `UI-012` `[R]` Start/End period controls are **month pickers** (display "Oct-26"), not day pickers (`DASH-003`). Region and branch filters (Daily Upload) offer "All".
- `UI-013` `[C]` **Tooltips** (`DASH-001`): on hover, keyboard focus and touch, every bar, card, growth point and grid cell shows target, real and % (growth: also last-year and growth %). Tooltips are reachable without a mouse, do not trap focus, and do not cover the element's own label. Content contract: `01-metrics-contract.md` section 7.
- `UI-014` `[R]` Growth matrix: 5 region rows x 7 month columns, paired bars (last year, this year), growth line and arrow labels; month names rather than numbers; year shown per bar; horizontal scroll only inside the matrix on narrow screens.
- `UI-015` `[R]` Daily Upload grid: sticky distributor column and day header; green Upload / orange Not Upload cells with a text or icon cue; future days blank; row count of upload days.
- `UI-016` `[P]` Brand preset "Inafood dark" reproduces the Tableau look (navy gradient, orange outlines, logo); logo and backgrounds come from approved assets (the workbook contains `Logo Inafood.png` and background images; confirm usage rights before committing them).

### Theme and responsive
- `UI-008` `[C]` Light, dark and system modes using shared design tokens.
- `UI-009` `[C]` Current stable Chrome and Firefox; responsive on smaller screens; no needless page-wide horizontal scroll.
- `UI-010` `[P]` Presentation/kiosk mode maximizes chart space and hides editing controls.
- `UI-011` `[C]` Honor reduced-motion; readable contrast in both themes.

### Dashboard editor
- `EDIT-001` `[C]` Separate View and Edit modes. View: chart interactions on, layout drag/resize off.
- `EDIT-002` `[C]` Grid drag/drop, resize, snapping, spacing, responsive breakpoints.
- `EDIT-003` `[C]` Widget registry: KPI card, bar chart, text/heading, image/background; later widgets plug into the same registry.
- `EDIT-004` `[C]` Add, remove, duplicate, move, resize, reorder, rename, configure widgets.
- `EDIT-005` `[C]` Text supports safe limited formatting; never unsanitized HTML.
- `EDIT-006` `[P]` Background controls: approved asset, fit/position, overlay/opacity, contrast. Suggest a curated asset library in v1 instead of arbitrary uploads.
- `EDIT-007` `[C]` Save, preview, cancel/revert, unsaved-change warning, versioned layout, validation.
- `EDIT-008` `[C]` Layouts store typed JSON only, never code or SQL.
- `EDIT-009` `[P]` Concurrent saves use optimistic versioning; conflicts prompt reload/compare.

### Dashboard rotation
- `PLAY-001` `[C]` Admin selects dashboards, order, enabled state, per-item duration; default 10 s.
- `PLAY-002` `[P]` Enforce a minimum duration (proposed 5 s).
- `PLAY-003` `[C]` Controls: start, pause, resume, next, previous, exit.
- `PLAY-004` `[C]` One controller/state machine; timers and listeners cleaned up; no overlapping intervals.
- `PLAY-005` `[C]` No rotation while editing or with unsaved changes.
- `PLAY-006` `[C]` Visible, non-blocking behavior on dashboard load failure (skip or keep last good, by setting); never a blank screen.
- `PLAY-007` `[O]` Whether filters persist across rotation or each dashboard keeps its own (DEC-010).
- `PLAY-008` `[C]` Playlists persist server-side once storage is approved.
- `PLAY-009` `[C]` **The CEO manages what rotates and for how long.** A user with `display.manage` on a display (for example the CEO for the CEO TV) chooses which dashboards are in that display's playlist, their order, and the duration of each. Changes reach the TV within seconds, without reloading it. Admins grant this right per user and per display (`PLAY-001` stays the admin default).
- `PLAY-010` `[C]` Each item can carry a **presentation recommendation**: `half`, `full` or `auto`. It is advisory: a web page cannot resize the browser window, and the user switches the dashboard window between half and full by hand (DEC-078). The recommendation appears on the remote-control screen and as a small hint on the TV (for example "best in full screen"); the layout itself always follows the actual window size (`UI-041`).
- `PLAY-011` `[P]` A **display** is a named screen with its own revocable credential and its own playlist (DEC-079). Several displays can exist, each with its own playlist.
- `PLAY-012` `[P]` **Remote control** from a phone or laptop: next, previous, pause, resume, jump to a dashboard, change a duration; the TV follows.
- `PLAY-013` `[P]` Dashboards in a playlist that the display's user is not allowed to see are skipped and reported to the manager, never shown.

### Achievement colors
- Configure thresholds and colors in admin settings, never hardcoded in chart components. Provide preview, restore defaults, safe fallback, validation of ordering/range/contrast, audit/version history where practical. Rule itself: `01-metrics-contract.md` section 4.

## 3. Layout zones

1. **Sidebar:** collapsible navigation, dashboard groups, current selection.
2. **Top bar:** title/breadcrumb, global filters, theme control, refresh/freshness indicator, authorized actions.
3. **Canvas:** responsive grid of registered widgets.
4. **Edit toolbar (edit mode):** add widget, undo/cancel, save/publish, unsaved status.
5. **Presentation controls:** unobtrusive pause/resume, next/previous, exit, optional time remaining.
6. **Admin settings:** achievement rules, playlists, layout permissions, approved assets.

Not every dashboard uses every zone; the shell stays consistent.

## 4. Visual tokens and typography

Define tokens centrally; no per-component hardcoded colors.

- Canvas, surface, elevated surface, border, primary/secondary/muted text, focus ring.
- Brand/accent, success, warning, danger, info.
- Chart series, gridlines, tooltip surfaces, selected-tab, hover/focus states.
- Spacing scale, radius, elevation, font families/sizes/line heights, chart label sizes.

Neutral professional base with a restrained accent. The screenshot's orange/blue may be an optional brand preset but must not dictate all components or reduce contrast. Light and dark both use semantic tokens.

## 5. Responsive behavior

- CSS grid/flex and breakpoints; no fixed pixel placement of the whole dashboard.
- Container-aware chart resize after sidebar, grid, theme or viewport changes.
- Narrow widths: stack/reflow; never shrink labels to unreadable sizes.
- Table needing horizontal scroll scrolls inside its own region only.
- Target displays are defined in section 10: the CEO TV is 4K with the dashboard on **half** of the screen (the other half shows CCTV) or on the **full** screen. Test the four viewports in that table from Phase 1.
- Support browser zoom and high-DPI.
- Rotation must not cause layout jumps between dashboards with different widget counts.

## 6. Interaction states

Every interactive component defines: default, hover, focus-visible, pressed/selected, disabled, loading, empty, error, success (where applicable); keyboard navigation and focus restore after dialogs; confirmation for destructive actions (remove widget, delete playlist); undo or cancel in editing flows; toasts that do not steal focus or vanish before reading; draft state or confirmation before publishing shared configuration.

## 7. Chart UX

- Tooltips: series, region, period, actual/target (where approved), formatted achievement.
- Consistent axis units, number formatting, labels, legends.
- Never rely on red/green alone: add text, symbols or accessible descriptions (`RULE-007`).
- No clipped labels or overlapping bars on small containers.
- Provide a table or text summary for screen readers.
- Respect reduced motion; avoid animated transitions during fast rotation.
- Chart colors follow the theme and the configured rules.

## 8. Accessibility

- `A11Y-001` `[C]` Target WCAG 2.2 AA: keyboard access, visible focus, semantic labels, contrast.
- `A11Y-002` `[C]` Chart data alternatives (table/summary) for every chart.
- `A11Y-003` `[C]` Region tabs and selected state operable by keyboard and announced.
- `A11Y-004` `[C]` Loading placeholders never presented as real zero values.
- `A11Y-005` `[P]` Automated axe checks in CI plus manual keyboard and screen-reader review before release.

## 9. Language and formats (DEC-013)

- `I18N-001` `[C]` The interface supports **English and Bahasa Indonesia**.
- `I18N-002` `[C]` The **default language is English**. An admin can change the default language in settings (`ui.defaultLanguage`).
- `I18N-003` `[P]` A user can switch language from a toggle; the choice is saved on the user and overrides the default.
- `I18N-004` `[P]` Numbers, percentages and dates follow the language (English `1,234.5`; Indonesian `1.234,5`; month names localised). Formatting is display only: values stay exact `numeric` (`01-metrics-contract.md` section 11).
- `I18N-005` `[P]` All text comes from translation files (`en`, `id`), including chart labels, tooltips, glossary entries, empty and error states and export headers. A missing Indonesian key falls back to English; a CI check lists missing keys.
- `I18N-006` `[P]` Data names (regions, branches, distributors, SE) are shown as stored and never translated.
- `I18N-007` `[P]` The AI replies in the language of the user's question, otherwise in the user's selected language (`AI-017`).
- Time zone for "today" and the display date: single business time zone (DEC-006).

## 10. TV and presentation mode (CEO display)

The CEO TV is **4K** (3840 x 2160). Some dashboards use **half** of the screen (the other half shows CCTV); others use the **full** screen. The reference screenshots are the half-screen layouts (about 963 x 985). That matches a 4K TV running at 200% scaling: half screen is about **960 x 1080 CSS pixels**. This is an inference; the app reads the real window size at runtime, so the layout does not depend on it.

| Case | Physical area | Expected CSS viewport | Use |
|---|---|---|---|
| Half screen, 4K at 200% | 1920 x 2160 | about 960 x 1080 | Reference screenshots; most dashboards |
| Full screen, 4K at 200% | 3840 x 2160 | 1920 x 1080 | Wide dashboards (for example Sales Detail) |
| Half screen, 4K at 100% | 1920 x 2160 | 1920 x 2160 | Must still look right |
| Full screen, 4K at 100% | 3840 x 2160 | 3840 x 2160 | Must still look right |

- `UI-037` `[C]` **Half and full layouts.** Every dashboard works in half and full windows and in any size between. The app chooses its layout from the actual window size and aspect ratio. **Default: the dashboard window is half screen** next to the CCTV viewer window; the user can switch it to full screen by hand (then the CCTV cannot be seen). The Tableau screenshots are the visual target for half screen.
- `UI-038` `[P]` **TV mode** (presentation mode on a display): no sidebar or top bar, no scrollbars, mouse cursor hidden after a few seconds without movement and shown again when the mouse moves, large type readable from a distance (minimum sizes per window size), high contrast, little motion, no page reloads (data updates in place, `DASH-011`).
- `DASH-012` `[P]` **Persistent labels on the TV.** The TV is a mini PC with a mouse, so hover tooltips (`DASH-001`) work there too. But during unattended rotation nobody hovers, so TV mode also shows the values a tooltip would show (target, real, %) as **persistent on-screen labels** wherever space allows, with the same wording and numbers.
- `UI-039` `[C]` **Switching between half and full is done by the user** with the mouse (the TV is a mini PC with two windows: the CCTV viewer and the dashboard browser). The app does not try to resize or take over the screen; it reacts to the new window size immediately, without reload and without losing the playlist position.
- `UI-041` `[C]` **Auto-fit, no blank space.** The dashboard fills the **entire browser window** at any size (half, full, anything between): no white or empty margins, no letterboxing, no scrollbars in TV mode, background full-bleed (the page and window background use the theme color from the first paint, so there is no white flash). Layout reflows live when the window changes (half to full and back, or any resize). Primary method: responsive reflow, with chart cards that stretch to fill their space and type that scales with the window within a minimum and maximum. Fallback only for sizes outside the supported aspect range: uniform scale-to-fit with a matching background.
- `UI-042` `[P]` **TV setup guide** (kept in the repository): open the dashboard window in browser app mode (no tabs or address bar, for example Chrome `--app=<address>`) so the whole window is dashboard; keep the CCTV viewer in the other half; set the dashboard window as the display's default half.
- `UI-040` `[P]` Charts render sharply on 4K (device pixel ratio aware) and stay light enough for the TV's hardware.
