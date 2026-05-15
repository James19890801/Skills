# Resume/CV Generator Skill

An AI Skill that automatically generates professional, visually stunning HTML resumes from PDF/text/photo inputs, with Morandi color scheme, mobile-responsive design, PDF export functionality, and automatic deployment to Cloudflare Pages.

**Trigger phrases:** "help me make a resume", "optimize my resume", "create an online resume", "resume to HTML", "deploy my resume"

---

## Overview

```
Step 1: Collect raw materials (resume file + photo)
Step 2: Scan information completeness, proactively fill gaps
Step 3: Confirm supplementary content (max 2 key questions)
Step 4: Generate Morandi-style HTML resume
Step 5: Auto-deploy to Cloudflare Pages
Step 6: Open local preview
Step 7: Return online URL
```

---

## Step 1: Collect Raw Materials

**Proactively ask the user:**
1. Where is your original resume? (file path / paste text / screenshot)
2. Do you have a personal photo? If yes, where is it located?

Use `read_file` to read PDF/HTML resumes, or parse text content directly pasted by the user.

---

## Step 2: Information Completeness Scan (Mandatory)

After reading the resume, check each item on the following checklist. **Ask for anything missing**, do not skip:

### Basic Information Check
- [ ] Full name
- [ ] Phone number (if missing → proactively ask)
- [ ] Email (if missing → proactively ask)
- [ ] Current city
- [ ] Job target (position direction)
- [ ] Personal photo (if missing → proactively ask, inform user they can upload image files)

### Work Experience Check
- [ ] Does each experience have clear **company name, position, start/end dates**
- [ ] Does each experience have at least 3 work descriptions (if only 1, follow up)
- [ ] **Gap scan**: Are there gaps exceeding 6 months between experiences → If yes, prompt user to explain or supplement
- [ ] Is the most recent experience within the last 2 years (if gap exceeds 1 year, annotate or supplement)

### Education Background Check
- [ ] School name, major, degree, graduation date completeness

### Other Risk Alerts
- If multiple experiences but minimal descriptions → alert that this may affect presentation, suggest supplementing

---

## Step 3: Confirmation Phase (Max 2 Questions)

Before generating, confirm with the user:
1. "I plan to structure your resume like this: [brief description], any additional thoughts?"
2. If there are obvious gaps or missing information, explain directly and ask for supplementary content

**Do not over-ask. 2 questions max. If information is sufficient, generate directly.**

---

## Step 4: Generate Morandi HTML Resume

### Color Scheme (Strictly Enforced, No Changes)

```css
:root {
  --ink: #2e3a42;
  --ink-light: #3d4f5a;
  --accent: #4a6274;          /* Morandi smoky blue */
  --accent-warm: #b8a070;     /* Light gold */
  --accent-green: #5a8070;    /* Morandi green */
  --sidebar-bg: #4a6274;      /* sidebar background */
  --sidebar-text: #dce6ec;
  --sidebar-dim: #a8bec9;
  --metric-bg: #f0f4f2;
  --metric-text: #3d6458;
  --metric-border: #b0ccc4;
  --award-bg: #faf6ee;
  --award-border: #c8aa7a;
  --rule: #dde6e2;
}
body { background: #e8e4de; }           /* Morandi beige */
.resume { background: #faf9f7; }        /* Warm white */
```

**Strictly prohibited:** Using black (#000), red, or pure gray as primary colors.

### HTML Structure Specification

```html
<body>
  <!-- Floating PDF export button (fixed top-right) -->
  <div class="fab-wrap">
    <button class="fab-btn" onclick="exportPDF()">⬇ Export PDF</button>
  </div>

  <div class="resume">
    <!-- Left sidebar (Morandi smoky blue background) -->
    <aside class="sidebar">
      <!-- Photo (base64 embedded) + Name + Contact + Skills tags + Education + Languages -->
    </aside>

    <!-- Right main (Work experience + Key highlights) -->
    <main class="main">
    </main>
  </div>
</body>
```

### Required Features

**1. Photo Embedding** (if photo provided):
```python
# Read image and convert to base64
import base64
with open(photo_path, 'rb') as f:
    img_b64 = base64.b64encode(f.read()).decode()
# In HTML: <img src="data:image/jpeg;base64,{img_b64}">
```

**2. Mobile Responsive (iPhone)**:
```css
@media (max-width: 600px) {
  .resume { flex-direction: column; width: 100%; }
  .sidebar { width: 100%; padding: 24px 20px 20px; }
  .sidebar > div:first-child { display: flex; flex-direction: row; align-items: center; gap: 16px; }
  .contact-list { display: flex; flex-wrap: wrap; gap: 6px 14px; }
  .main { padding: 20px 16px 32px; }
}
```

**3. PDF Export Button (Print background colors must be preserved)**:
```css
.fab-wrap { position: fixed; top: 20px; right: 20px; z-index: 999; }
.fab-btn {
  background: #4a6274; color: #fff; border: none; border-radius: 8px;
  padding: 9px 16px; font-size: 13px; cursor: pointer;
  -webkit-appearance: none; appearance: none;  /* Required for iOS Safari */
}
.sidebar {
  print-color-adjust: exact;
  -webkit-print-color-adjust: exact;  /* Preserve background color when printing */
}
@media print { .fab-wrap { display: none !important; } }
```

```javascript
function exportPDF() {
  var btn = document.querySelector('.fab-wrap');
  btn.style.display = 'none';
  var style = document.createElement('style');
  style.id = 'print-override';
  style.innerHTML = `
    @page { size: A4; margin: 0; }
    @media print {
      html, body { width: 210mm; margin: 0 !important; padding: 0 !important; }
      * { print-color-adjust: exact !important; -webkit-print-color-adjust: exact !important; }
      .resume { width: 794px !important; flex-direction: row !important; box-shadow: none !important; }
      .sidebar { width: 236px !important; background: #4a6274 !important; color: #dce6ec !important; }
    }
  `;
  document.head.appendChild(style);
  window.print();
  setTimeout(function() {
    var s = document.getElementById('print-override');
    if (s) s.remove();
    btn.style.display = '';
  }, 1000);
}
```

### Content Writing Guidelines
- Job target: Only write **position direction**, do not list target company names
- Skill tags: Limit to **5-8**, focus on core competencies, no堆砌
- Work highlights with data: Extract quantitative metrics where possible (e.g., "reduced costs by 60%", "managed 87 warehouses")
- Preserve **original information completeness** for each work experience, do not over-compress

---

## Step 5: Deploy to Cloudflare Pages

After generating HTML, automatically execute deployment:

```powershell
# 1. Save HTML to Desktop
$htmlPath = "C:\Users\Administrator\Desktop\[Name]_Resume.html"

# 2. Sync to deployment directory
$deployDir = "C:\Users\Administrator\Desktop\cv-deploy"
Copy-Item $htmlPath "$deployDir\index.html" -Force

# 3. Deploy (project name uses pinyin or initials, e.g., zhangsan-cv)
npx wrangler pages deploy "$deployDir" --project-name=[name-pinyin]-cv --branch=main --commit-dirty=true
```

**Notes:**
- If `cv-deploy` directory doesn't exist, create it first and place `index.html` inside
- project-name can only use lowercase letters, numbers, hyphens
- After successful deployment, extract the URL `https://xxxxxxxx.[project-name].pages.dev` from output

---

## Step 6: Open Local Preview

After deployment, automatically open local HTML preview in browser:

```powershell
Start-Process $htmlPath
```

---

## Step 7: Return Results

Report to user:
1. Local file path: `C:\Users\...\[Name]_Resume.html`
2. Online URL: `https://xxxxxxxx.[project-name].pages.dev`
3. Usage instructions: Top-right floating button → Click "Export PDF" → System print dialog → Select "Save as PDF" → A4 format

---

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| iOS Safari button displays black | Must add `-webkit-appearance: none` |
| Sidebar background color disappears when printing | Add `print-color-adjust: exact` to sidebar + add `* {print-color-adjust: exact !important}` in JS injection + hardcode sidebar background color |
| wrangler deployment fails | Check if Node.js is installed; check if `npx wrangler login` is logged in |
| Photo read failure | Confirm path is correct; supports jpg/png/webp formats |
| Gap exceeds 6 months | Label as "Career Development Period" in corresponding timeframe or ask user if they want to supplement |

---

## Robustness Assessment

### Strengths
1. **Complete workflow**: 7-step end-to-end automation from input to deployment
2. **Visual quality**: Morandi color scheme with strict CSS enforcement
3. **Cross-platform**: Mobile responsive + PDF export + print background preservation
4. **Error handling**: Information completeness scan with gap detection

### Areas for Improvement
1. **Photo handling**: Currently requires manual path input; could support drag-and-drop
2. **Multi-language**: Currently Chinese-focused; English version templates needed
3. **Template variety**: Currently single layout; could offer 2-3 style options
4. **Deployment dependency**: Requires wrangler login; could add fallback to GitHub Pages

### Production Readiness: 8/10
- Core functionality solid and tested
- Visual design professional and distinctive
- Deployment pipeline reliable
- Minor UX improvements needed for broader adoption
