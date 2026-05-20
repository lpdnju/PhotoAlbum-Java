---
name: generate-report-html
description: Generate report.html from assessment report data (report.json or js-assessment-report.md) and facts/ directory
---

# Generate Report HTML

Generate a self-contained `report.html` file from the assessment output. This skill reads the report data and AI-generated fact documents, then produces an interactive HTML report with the same structure and styling as the local CLI renderer.

## Input Parameters

- `workspace-path` (optional): Path to the project (defaults to current directory)

## Execution Steps

### Step 1: Locate the Report Directory

Find the versioned report directory at `.github/modernize/assessment/reports/report-{reportId}/`.
The `reportId` is the timestamp-based directory name (e.g., `report-20240101120000`).

### Step 2: Determine Language Type

Check which report file exists in the report directory:
- If `report.json` exists → Java/.NET project (AppCAT report)
- If `js-assessment-report.md` exists → JavaScript/TypeScript project (NCU report)

### Step 3: Read Input Data

**For Java/.NET (report.json):**
- Parse `report.json` to extract:
  - `components` array → application info (name, language, frameworks, buildTools, properties)
  - `incidents` per component → issues grouped by ruleId
  - `rules` object → rule titles, severities, descriptions, labels (domain=cloud-readiness, domain=*-upgrade)
  - `metadata.targetDisplayNames` and `metadata.targetIds` → target compute services
  - `security` array (if present) → security findings with id, title, severity, storyPoint, files, description
  - `rearchitectFindings` (if present) → framework obsoletion findings
- Read fact files from the `facts/` subdirectory (architecture-diagram.md, dependency-map.md, api-service-contracts.md, data-architecture.md, configuration-inventory.md, business-workflows.md)

**For JavaScript/TypeScript (js-assessment-report.md):**
- Parse the NCU (npm-check-updates) markdown output:
  - Lines starting with "Using " → package manager
  - Category headers: "Patch", "Minor", "Major", "Major Version Zero"
  - Dependency lines contain `→` arrow between current and target versions
- Read fact files from the `facts/` subdirectory

### Step 4: Generate report.html

Write the file to `.github/modernize/assessment/reports/report-{reportId}/report.html`.

**IMPORTANT:** Do NOT include a "Back to aggregate dashboard" link. The report must be self-contained.

---

## HTML Structure for Java/.NET Reports

The HTML must include these sections in order:

### 1. HTML Head
```html
<!DOCTYPE html>
<html><head><meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Assessment - {ApplicationName}</title>
```

### 2. CSS (embedded in `<style>` tag)

Use these CSS variables:
```css
:root {
  --bg-primary: #ffffff;
  --bg-card: #f8f9fa;
  --bg-page: #f0f1f3;
  --text-primary: #24292f;
  --text-secondary: #57606a;
  --text-muted: #6b7280;
  --border-color: #d0d7de;
  --border-light: #e1e4e8;
  --link-color: #2563eb;
  --color-mandatory: #E3008C;
  --color-potential: #637CEF;
  --color-optional: #A19F9D;
  --font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, Arial, sans-serif;
  --font-mono: 'Cascadia Code', 'Consolas', 'Courier New', monospace;
}
```

Key CSS classes:
- `.main` - page container with 24px 32px padding
- `.report-card` - card with #f8f9fa background, 1px border, 4px border-radius
- `.app-info-container` - flex two-column layout for app info
- `.domain-summary-container` - flex container for donut charts
- `.issue-table` - full-width collapsed table with fixed layout
- `.issue-row` - clickable row with cursor:pointer
- `.detail-row` - hidden by default (display:none), shows file list + explanation
- `.crit-label` - inline-flex with colored square + text
- `.expand-btn` - 20px button that rotates 90deg when open
- `.filter-bar` - flex container for domain/criticality multi-select dropdowns
- `.badge-experimental` - yellow badge for AI-generated content

### 3. Body Structure

```html
<body>
<div class="main">
  <h1>{ApplicationName}</h1>

  <!-- Application Information card -->
  <div class="report-card">
    <h2>Application Information</h2>
    <!-- Two-column layout: left (Name, Java Version if present, Effort), right (Build Tools, Frameworks) -->
  </div>

  <!-- Tab navigation (Issues tab + fact tabs) -->
  <div class="tab-nav">...</div>

  <!-- Issues tab panel -->
  <div id="tab-issues" class="tab-panel active">

    <!-- Target Compute Service dropdown (if targets present) -->
    <!-- Issue Summary card with SVG donut charts per domain -->
    <!-- Filter bar (Domain + Criticality multi-selects) -->
    <!-- Cloud Readiness issue table -->
    <!-- Upgrade issue table (includes rearchitect findings) -->
    <!-- Security issue table (with Experimental badge) -->

  </div>

  <!-- Fact tab panels (one per available fact file) -->

  <!-- Footer with feedback link -->
  <p class="footer">Your feedback is invaluable — <a href="https://aka.ms/ghcp-appmod/feedback">Share feedback</a></p>
</div>
```

### 4. Issue Tables Structure

Each issue section:
```html
<div class="issue-section" data-domain="{domainId}">
  <h2>{Section Title}</h2>
  <table class="issue-table">
    <colgroup><col class="col-issue"/><col class="col-criticality"/><col class="col-storypoint"/></colgroup>
    <thead><tr><th>Issue Category</th><th>Criticality</th><th>Story Point</th></tr></thead>
    <tbody>
      <!-- For each rule group, ordered by severity (mandatory first) then by count: -->
      <tr class="issue-row" data-criticality="{severity}" data-targets='{json}' onclick="toggleRow('{rowId}', this)" style="cursor:pointer;">
        <td style="white-space:normal;"><div class="issue-title-cell"><button class="expand-btn" id="btn-{rowId}">&#x276F;</button> {title}</div></td>
        <td class="crit-cell"><span class="crit-label"><span class="crit-square crit-square-{severity}"></span>{CritText}</span></td>
        <td class="sp-cell">{storyPoint}</td>
      </tr>
      <tr class="detail-row" id="{rowId}" style="display:none;">
        <td colspan="3">
          <div class="detail-content">
            <div class="file-list"><!-- table with File and Position columns --></div>
            <div class="explanation-panel"><!-- rule description --></div>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

### 5. SVG Donut Charts

For each domain with issues, render an SVG donut chart:
- ViewBox: 0 0 100 100, width/height: 100
- Circle radius: 35, stroke-width: 15
- Transform: rotate(-90deg) for start-at-top
- Segments use stroke-dasharray with 1.5px gap between segments
- Colors: mandatory=#E3008C, potential=#637CEF, optional=#A19F9D

### 6. Effort Label

Calculate total effort = component.Effort + security storyPoints + rearchitect count * 10
- < 20 → "S"
- < 50 → "M"
- < 100 → "L"
- >= 100 → "XL"

Display as: "{Label} (total story points: {total})"

### 7. Domain Classification

For each rule, check its `labels` array:
- `domain=cloud-readiness` → Cloud Readiness section
- Label ending with `-upgrade` → Upgrade section (e.g., "domain=java-upgrade")
- For .NET: rules without a domain label default to Cloud Readiness
- Security findings → Security section

### 8. JavaScript (embedded in `<script>` tag)

Required functions:
- `toggleRow(rowId, triggerRow)` - show/hide detail rows
- `toggleMultiSelect(id)` - open/close filter dropdowns
- `getMultiSelectValues(msId)` - get checked filter values
- `onMultiSelectChange(msId, label)` - update button text, apply filters
- `applyFilters()` - show/hide rows based on domain + criticality selections
- `clearAllFilters()` - reset all checkboxes
- `onTargetChange(targetId)` - recalculate effort/severity per target, update donuts
- `buildDonutSvg(m, p, o)` - generate SVG string for donut chart
- `updateDonuts(domainCrits)` - rebuild donut container from new counts

### 9. Fact Tabs

For each available fact file in `facts/`, create a tab:
- Tab IDs: `tab-architecture-diagram`, `tab-dependency-map`, etc.
- Tab labels: "Architecture", "Dependencies", "API Contracts", "Data", "Configuration", "Workflows"
- Content: Convert the fact markdown to HTML (render Mermaid code blocks as `<pre class="mermaid">`)
- Include Mermaid.js CDN script for diagram rendering

---

## HTML Structure for JavaScript/TypeScript Reports

Simpler structure (no donut charts, no filtering, no per-target):

```html
<body>
<div class="main">
  <h1>{RepoName} — JavaScript/TypeScript Dependency Assessment</h1>

  <!-- Summary cards (Total, Patch, Minor, Major, Major 0.x) -->
  <div class="summary-cards">...</div>

  <!-- Tab navigation (Dependency Updates + fact tabs) -->

  <!-- Dependency Updates tab -->
  <table class="dep-table">
    <thead><tr><th>Package</th><th>Current</th><th>Target</th><th>Type</th></tr></thead>
    <tbody>
      <!-- For each dependency: -->
      <tr><td>{name}</td><td>{current}</td><td>{target}</td><td><span class="badge badge-{type}">{type}</span></td></tr>
    </tbody>
  </table>

  <!-- Recommendations section -->

  <!-- Fact tab panels -->

  <!-- Footer -->
</div>
```

Badge colors for JS/TS:
- Patch: green (#22c55e bg)
- Minor: blue (#3b82f6 bg)
- Major: orange (#f97316 bg)
- Major (0.x): red (#ef4444 bg)

---

## Rearchitect Findings (appended to Upgrade section)

For each rearchitect finding, add rows to the Upgrade table:
```html
<tr class="issue-row" data-criticality="optional" onclick="toggleRow('{rowId}', this)">
  <td>Framework obsoletion ({finding.old})</td>
  <td><span class="crit-label"><span class="crit-square crit-square-optional"></span>Optional</span></td>
  <td class="sp-cell">10</td>
</tr>
<tr class="detail-row" id="{rowId}" style="display:none;">
  <!-- file list from detectedIn.configFiles + detectedIn.sourceFiles -->
  <!-- explanation panel with finding.explanation -->
</tr>
```

Default story points for rearchitect findings: **10**

---

## Success Criteria

1. `report.html` is written to `.github/modernize/assessment/reports/report-{reportId}/report.html`
2. The HTML is self-contained (all CSS/JS inline, no external dependencies except Mermaid CDN)
3. No "Back to aggregate dashboard" link is present
4. All issues from report.json are rendered with correct domain classification
5. Security findings are marked with "Experimental" badge
6. Fact tabs are present for each available fact file in the facts/ directory
7. Interactive features work: expand/collapse rows, domain/criticality filters, target switching
