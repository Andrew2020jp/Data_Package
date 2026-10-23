# 0016. Design system: Material Design + Bundestag corporate look

- **Status:** Proposed (light theme + first components shipped; full system is the target)
- **Date:** 2026-06-14
- **Deciders:** maintainer (ma3u)
- **Source(s):** `web/index.html`, `web/aggregate.html` (`<style>` token blocks); ADR-0015
  (self-hosted Roboto); redesign commits `3551f98`, `197bb2b`

## Context

The Pages app started as a GitHub-dark UI. For a public, jury-facing parliamentary tool the
look should read as **Material Design** (clear, modern, accessible) **in a Bundestag corporate
feel** (the institution's blue, restraint, trust). A first pass shipped a light theme with a blue
app bar + white elevated cards, but it is not yet a *consistent, documented* design system, and
not yet visibly "Material 3" or authentically Bundestag-CD. This ADR fixes the **direction and
tokens** so future UI work is consistent rather than ad-hoc.

## Decision

Adopt one design system across `index.html` and `aggregate.html`: **Material Design (M3
principles) expressed in Bundestag corporate-design cues**, light theme.

**Design tokens (single source of truth in each page's `:root`; names kept stable so JS inline
styles keep working):**
- **Color roles** — `--accent` Bundestag-Blau `#16467f` (primary) · `--accent-d`/`--accent-l`
  variants · surfaces `--panel` `#fff` / `--panel-2` `#f6f8fb` · background `--bg` `#eef1f5` ·
  text `--text` `#1a2330` / `--muted` · outline `--border` · semantic
  `--ok/--warn/--err/--neutral`. **Selection/accent-gold** `#f2c14e` (graph highlight) as a
  Bundestag-adjacent secondary.
- **Graph canvas** stays deep navy `--graph-bg` `#0b2239` (plenar-feel; keeps colored nodes
  legible — light bg would wash them out).
- **Elevation** — `--e1` (cards) / `--e2` (app bar, modals) soft blue-tinted shadows (Material
  elevation, not borders).
- **Shape** — cards/modals 12–16px radius; buttons/chips pill (16–20px).
- **Type** — **Roboto, self-hosted** (ADR-0015); section labels uppercase + letter-spacing.
- **Motion** — short transitions on interactive surfaces (state layers).

**Components:** blue **app bar** (elevation, white text) · pill **tabs/buttons** with hover
state layer (filled-on-blue in the bar, tonal in the body, `.active` = filled) · elevated
**cards** · **modal** video player · selection = gold node + dimmed neighbourhood.

## Consequences

- Consistent visual language across both pages; new UI reuses the tokens.
- Graph stays dark by design (documented), the rest is light Material.
- **Open work toward the full target** (tracked in
  `docs/planning/future/design-system-rollout.md`): verify the *official* Bundestag CD blue/gold
  values; a discreet **Bundestagsadler/wordmark** in the app bar; a proper **Material type
  scale**; **state-layer/ripple** affordances + focus rings; optional **FAB** for the primary
  action; motion polish; WCAG-AA contrast audit of the light theme.

## Stand 2026-06-21 (Rollout)

Umgesetzt: Kuppel-Marke + Wortmarke, Gold-Akzent, Type-Scale (begonnen), State-Layer + Fokus-Ringe
flächendeckend, **FAB** „Video der Sitzung", **Motion** (Modal-Scale-in, reduced-motion), **WCAG-AA
0 Verstöße** (axe-core). **CD-Blau** auf `#0b3e7a` gesetzt — **ausdrücklich eine Näherung**: die
offizielle Bundestag-CD-Spezifikation (HKS/Pantone) ließ sich nicht belegen (Web-Suche + Live-Probe
ohne Treffer). Die Farbe liegt zentral als Token vor und ist bei Vorlage des CD-Handbuchs 1:1
austauschbar — sie ist **nicht** als amtlich verbürgt zu verstehen.

## Alternatives considered

- **Dark theme** — rejected: Material 3 + Bundestag CD are light; jury readability favors light.
- **A CSS framework (MUI/Material Web)** — rejected: the app is a single static file with no
  build step (Pages, dep-free CI); hand-rolled tokens keep it buildless and small.
- **Ad-hoc styling per page** — rejected: drift; this ADR makes the tokens the contract.
