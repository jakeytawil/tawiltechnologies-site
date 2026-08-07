# Demo Access Gate Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Visitors to tawiltechnologies.com submit name + email in an elegant dialog and are shown (and emailed) the demo link tawiltech.netlify.app; every request emails a lead to jacob@tawiltechnologies.com.

**Architecture:** Pure static-site change — new `<dialog>` + vanilla JS `fetch` to FormSubmit's AJAX endpoint in `index.html`; site-wide email address swap. No backend, no build step, no new files outside docs.

**Tech Stack:** Plain HTML/CSS/JS, FormSubmit (formsubmit.co AJAX endpoint), GitHub Pages.

**Spec:** `docs/superpowers/specs/2026-08-07-demo-access-gate-design.md`

## Global Constraints

- Static site, no build step, no framework, no new runtime dependencies.
- All work in repo `C:\Users\jakey\Downloads\tawiltechnologies-site`, branch `main`.
- Lead + form email address everywhere: `jacob@tawiltechnologies.com` (replaces `jakeytawil18@gmail.com`; zero occurrences of the old address may remain in html files).
- Demo URL: `https://tawiltech.netlify.app` (open in new tab).
- Visual language: existing tokens only (`--pine`, `--brass`, `--ivory`, `--line`, Fraunces headings, Hanken Grotesk body). Match the existing `#contactDlg` structure/classes.
- Copy strings (verbatim):
  - Button CTA: `Request private access`
  - Dialog heading: `Request a private viewing`
  - Dialog sub-line: `Access is granted personally. Tell us who you are and your invitation will be on its way.`
  - Submit button: `Request access`
  - Success heading: `Welcome — your access is ready.`
  - Success button: `Enter the platform →`
  - Error line: `Something went wrong — please try again, or write us at jacob@tawiltechnologies.com.`
  - Lead email subject: `Demo access requested — tawiltechnologies.com`
- No test framework exists; each task verifies in a real browser (Playwright MCP tools) against the local file, then commits. The live FormSubmit happy path is a manual post-deploy step (it sends real email and triggers one-time activation).

---

### Task 1: Site-wide email address swap

**Files:**
- Modify: `index.html:157` (form action), `index.html:165` (mailto + text)
- Modify: `privacy.html:106`
- Modify: `terms.html:90`

**Interfaces:**
- Produces: all three pages reference only `jacob@tawiltechnologies.com`. Task 2/3 write new markup that uses this address from the start.

- [ ] **Step 1: Replace every `jakeytawil18@gmail.com` with `jacob@tawiltechnologies.com`**

In each of `index.html`, `privacy.html`, `terms.html`, replace all occurrences (form `action` URL, `mailto:` hrefs, visible link text). Example for `index.html:157`:

```html
<form action="https://formsubmit.co/jacob@tawiltechnologies.com" method="POST">
```

and `index.html:165`:

```html
<p class="dlg-alt">Or write directly: <a href="mailto:jacob@tawiltechnologies.com">jacob@tawiltechnologies.com</a></p>
```

- [ ] **Step 2: Verify zero old references remain**

Run: `grep -rn "jakeytawil18" *.html`
Expected: no output (exit code 1).

- [ ] **Step 3: Commit**

```bash
git add index.html privacy.html terms.html
git commit -m "chore: switch contact email to jacob@tawiltechnologies.com"
```

---

### Task 2: Access dialog markup, buttons, and CSS

**Files:**
- Modify: `index.html` — CSS block (after `.btn-sm` rule ~line 35), hero (~line 108), close section (~line 141), new `<dialog>` before the `<script>` tag (~line 168)

**Interfaces:**
- Consumes: existing classes `.btn`, `.eyebrow`, `.dlg`, `.dlg-head`, `.dlg-sub`, `.field`, `.dlg-alt`; existing dialog pattern from `#contactDlg`.
- Produces (Task 3 relies on these exact ids): `#accessDlg`, `#accessForm`, `#a-name`, `#a-email`, `#a-portfolio`, `#a-company` (honeypot), `#a-submit`, `#a-err`, `#accessFormWrap`, `#accessGranted`. Global function name `openAccess()` referenced by both buttons' `onclick`.

- [ ] **Step 1: Add CSS for the outline button, honeypot, error line, and granted state**

Insert after the `.btn-sm` rule:

```css
.btn-outline{background:transparent;color:var(--ink);border:1px solid var(--brass)}
.btn-outline:hover{background:var(--pine);color:var(--ivory)}
.cta-row{display:flex;gap:12px;justify-content:center;flex-wrap:wrap}
.hp{position:absolute;left:-9999px;width:1px;height:1px;overflow:hidden}
.form-err{font-size:13px;color:#a03c2e;margin-top:2px}
.granted{text-align:center;padding:6px 0 2px}
.granted h3{font-size:24px;font-weight:500;margin:8px 0 6px}
.granted p{font-size:13.5px;color:var(--ink-soft);margin-bottom:18px}
```

- [ ] **Step 2: Add the CTA buttons**

Hero — replace the single button line (`<button class="btn" onclick="openContact()">Contact us</button>` inside `<header class="hero">`) with:

```html
  <div class="cta-row">
    <button class="btn btn-outline" onclick="openAccess()">Request private access</button>
    <button class="btn" onclick="openContact()">Contact us</button>
  </div>
```

Close section — replace its `<button class="btn" onclick="openContact()">Contact us</button>` with the same `cta-row` block (identical markup).

- [ ] **Step 3: Add the access dialog before the `<script>` tag**

```html
<dialog id="accessDlg" aria-label="Request private access">
  <div class="dlg">
    <div id="accessFormWrap">
      <div class="dlg-head"><div><span class="eyebrow">Tawil Technologies</span><h3>Request a private viewing</h3></div><button type="button" onclick="document.getElementById('accessDlg').close()" aria-label="Close">×</button></div>
      <p class="dlg-sub">Access is granted personally. Tell us who you are and your invitation will be on its way.</p>
      <form id="accessForm">
        <div class="field"><label for="a-name">Name</label><input id="a-name" name="name" required autocomplete="name" /></div>
        <div class="field"><label for="a-email">Email</label><input id="a-email" name="email" type="email" required autocomplete="email" /></div>
        <div class="field"><label for="a-portfolio">Your portfolio</label><textarea id="a-portfolio" name="portfolio" placeholder="e.g. 120 units across four buildings"></textarea></div>
        <div class="hp" aria-hidden="true"><label for="a-company">Company</label><input id="a-company" name="company" tabindex="-1" autocomplete="off" /></div>
        <p class="form-err" id="a-err" hidden>Something went wrong — please try again, or write us at jacob@tawiltechnologies.com.</p>
        <button class="btn" type="submit" id="a-submit">Request access</button>
      </form>
    </div>
    <div id="accessGranted" hidden>
      <div class="dlg-head"><div><span class="eyebrow">Tawil Technologies</span></div><button type="button" onclick="document.getElementById('accessDlg').close()" aria-label="Close">×</button></div>
      <div class="granted">
        <h3>Welcome — your access is ready.</h3>
        <p>A copy has been sent to your email.</p>
        <a class="btn" href="https://tawiltech.netlify.app" target="_blank" rel="noopener">Enter the platform →</a>
      </div>
    </div>
  </div>
</dialog>
```

- [ ] **Step 4: Add a temporary stub so buttons don't throw before Task 3**

Inside the existing `<script>` block, add:

```js
const adlg = document.getElementById('accessDlg');
function openAccess(){ adlg.showModal(); document.getElementById('a-name').focus(); }
adlg.addEventListener('click', e => { if (e.target === adlg) adlg.close(); });
```

- [ ] **Step 5: Verify in browser**

Open `file:///C:/Users/jakey/Downloads/tawiltechnologies-site/index.html` (Playwright `browser_navigate` + `browser_snapshot`).
Expected: both sections show the two buttons side by side; clicking **Request private access** opens the dialog with Name/Email/Your portfolio fields; honeypot field not visible; close button and backdrop click both close it; layout intact at 375px width (`browser_resize`).

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: request-private-access dialog and CTAs"
```

---

### Task 3: Submit behavior — FormSubmit AJAX, honeypot, success/error states

**Files:**
- Modify: `index.html` — the `<script>` block only

**Interfaces:**
- Consumes: ids from Task 2 (`#accessForm`, `#a-name`, `#a-email`, `#a-portfolio`, `#a-company`, `#a-submit`, `#a-err`, `#accessFormWrap`, `#accessGranted`), Task 1 address.
- Produces: form submits to `https://formsubmit.co/ajax/jacob@tawiltechnologies.com`; success swaps to granted state; failure shows `#a-err` and re-enables submit; filled honeypot skips the network call but still shows granted.

- [ ] **Step 1: Extend the script**

Replace the Task 2 stub's `openAccess` addition so the full script block reads:

```html
<script>
const dlg = document.getElementById('contactDlg');
function openContact(){ dlg.showModal(); document.getElementById('f-name').focus(); }
dlg.addEventListener('click', e => { if (e.target === dlg) dlg.close(); });

const adlg = document.getElementById('accessDlg');
function openAccess(){ adlg.showModal(); document.getElementById('a-name').focus(); }
adlg.addEventListener('click', e => { if (e.target === adlg) adlg.close(); });

function showGranted(){
  document.getElementById('accessFormWrap').hidden = true;
  document.getElementById('accessGranted').hidden = false;
}

document.getElementById('accessForm').addEventListener('submit', async e => {
  e.preventDefault();
  const btn = document.getElementById('a-submit');
  const err = document.getElementById('a-err');
  err.hidden = true;
  if (document.getElementById('a-company').value){ showGranted(); return; }
  btn.disabled = true; btn.textContent = 'Requesting…';
  try {
    const res = await fetch('https://formsubmit.co/ajax/jacob@tawiltechnologies.com', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
      body: JSON.stringify({
        _subject: 'Demo access requested — tawiltechnologies.com',
        _template: 'table',
        _captcha: 'false',
        _autoresponse: 'Welcome — here is your private access to the Tawil platform: https://tawiltech.netlify.app\n\nWe would love to hear what you think — reply to this email any time.\n\n— Jacob Tawil, Tawil Technologies',
        name: document.getElementById('a-name').value,
        email: document.getElementById('a-email').value,
        portfolio: document.getElementById('a-portfolio').value
      })
    });
    if (!res.ok) throw new Error('send failed');
    showGranted();
  } catch {
    err.hidden = false;
  } finally {
    btn.disabled = false; btn.textContent = 'Request access';
  }
});
</script>
```

- [ ] **Step 2: Verify honeypot path (no network call)**

Playwright: navigate to the local file, `browser_evaluate` to set `#a-company` value to `bot`, fill name/email, submit, then `browser_network_requests`.
Expected: granted state visible (`Welcome — your access is ready.` + `Enter the platform →` linking to `https://tawiltech.netlify.app`), and **no** request to `formsubmit.co` in the network log.

- [ ] **Step 3: Verify error path (no reveal on failure)**

Playwright `browser_evaluate`: `window.fetch = () => Promise.reject(new Error('offline'))`, then fill name + email, submit.
Expected: error line visible with the exact copy, form still shown, submit button re-enabled with text `Request access`, granted state NOT visible.

- [ ] **Step 4: Verify success path with stubbed fetch (no real email)**

Playwright `browser_evaluate`: `window.fetch = () => Promise.resolve(new Response('{}', {status: 200}))`, reload first to clear prior stubs, fill, submit.
Expected: granted state visible; link opens tawiltech.netlify.app in a new tab.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: gate demo link behind access request (FormSubmit AJAX)"
```

---

### Task 4: Deploy and FormSubmit activation

**Files:** none (git push + manual browser steps)

**Interfaces:**
- Consumes: Tasks 1–3 committed on `main`.

- [ ] **Step 1: Push to deploy**

```bash
git push origin main
```

GitHub Pages redeploys automatically; confirm the new button appears on https://tawiltechnologies.com (may take 1–2 min; hard-refresh).

- [ ] **Step 2 (USER, manual): Activate FormSubmit for the new address**

On the live site, submit the access form once with real data (your own email). FormSubmit sends a one-time activation email to jacob@tawiltechnologies.com — open that mailbox and click the activation link. Also submit the existing **Contact us** form once (same address, same activation covers it — if a second activation email arrives, click it too).

- [ ] **Step 3 (USER, manual): Confirm end-to-end**

After activation, submit the access form again with real data. Expected: lead email arrives at jacob@tawiltechnologies.com with subject `Demo access requested — tawiltechnologies.com`; autoresponse with the demo link arrives at the submitted address; dialog shows the granted state.
