# Demo access gate — design spec

**Date:** 2026-08-07
**Site:** tawiltechnologies.com (static, GitHub Pages, repo `jakeytawil/tawiltechnologies-site`)
**Demo:** tawiltech.netlify.app (separate manual Netlify deploy — untouched by this work)

## Goal

Visitors must submit their name and email to get the demo link. The gate is lead
capture plus an invite-only feel — not access control. The demo URL is public and
that is accepted; the point is that a viewer clicks and identifies themselves
before the link is revealed.

No new infrastructure: no backend, no new repo, no email service. Everything is
an edit to the existing static pages, using FormSubmit (already in use for the
contact form).

## Scope

Files changed: `index.html`, `privacy.html`, `terms.html`. Nothing else.

## 1. On-site experience

- New CTA **Request private access** in two places: the hero and the closing
  ("See it on your portfolio") section, each alongside the existing "Contact us"
  button. Styled as the premium action (brass-outlined variant of `.btn`);
  "Contact us" keeps its current look.
- Clicking opens a new `<dialog>` in the existing visual language (Fraunces
  serif heading, brass eyebrow, ivory background — same structure as
  `#contactDlg`):
  - Heading: **Request a private viewing**
  - Sub-line: "Access is granted personally. Tell us who you are and your
    invitation will be on its way."
  - Fields: Name (required), Email (required, type=email), "Your portfolio"
    (optional textarea, placeholder e.g. "120 units across four buildings").
  - A visually hidden honeypot text field; if filled, no network call is made
    and the dialog shows the normal access-granted state (indistinguishable
    from success, so bots can't detect the trap; the link is public anyway —
    the honeypot only keeps junk out of the lead inbox).
- Submit button: **Request access**.

## 2. The unlock moment

- The form submits via `fetch` to FormSubmit's AJAX endpoint
  (`https://formsubmit.co/ajax/jacob@tawiltechnologies.com`) — the page never
  navigates.
- While in flight, the submit button shows a disabled/waiting state.
- **On success** the dialog content swaps to an access-granted state:
  - Eyebrow: "Tawil Technologies"
  - Heading: **Welcome — your access is ready.**
  - Brass button **Enter the platform →** linking to
    `https://tawiltech.netlify.app` (opens in a new tab).
  - Small line noting a copy has been sent to their email.
- **On failure** (network error or non-2xx): the dialog stays on the form,
  shows "Something went wrong — please try again, or write us at
  jacob@tawiltechnologies.com", and the button re-enables for retry. The demo
  link is **not** revealed without a successful submission.

## 3. Emails

Two emails per successful request, both via FormSubmit:

1. **Lead notification to jacob@tawiltechnologies.com** — subject set via
   `_subject`: "Demo access requested — tawiltechnologies.com"; body carries
   name, email, portfolio note.
2. **Autoresponse to the requester** — via `_autoresponse`: a short plain-text
   message containing the demo link so it sits in their inbox to return to.
   Plain text is a FormSubmit limitation and is acceptable; the real reveal
   happens on-site.

## 4. Email address change (site-wide)

Every `jakeytawil18@gmail.com` reference in `index.html`, `privacy.html`, and
`terms.html` changes to **jacob@tawiltechnologies.com**: the existing contact
form's action URL, all `mailto:` links, and any visible text. The new demo form
uses the new address from the start.

**Dependency (confirmed):** the jacob@tawiltechnologies.com mailbox exists and
receives mail. FormSubmit sends a one-time activation email to a newly used
address on first submission; until the link in it is clicked, submissions are
not delivered. First deploy therefore includes a manual step: submit each form
once and click the activation link(s).

## 5. Out of scope

- Real access control (passwords, unique tokens, revocation). The demo URL
  stays publicly reachable by anyone who has it.
- Any change to the Netlify demo site itself.
- Rate limiting beyond the honeypot (FormSubmit has its own abuse controls).
- Styled HTML autoresponse email (would require an email service + backend).

## 6. Testing

- Manual, against the live FormSubmit endpoint (it has no sandbox):
  - Happy path: submit with real data → dialog shows access-granted state,
    demo opens, lead email arrives at jacob@, autoresponse arrives at the
    submitted address.
  - Failure path: simulate offline / block the request → error message shows,
    form intact, retry works, no reveal.
  - Honeypot: fill hidden field → no network call, but access-granted state
    still shown; no lead email arrives.
  - Existing contact dialog still works after the address change (requires
    the one-time FormSubmit activation for the new address).
  - Mobile layout: dialogs usable at 375px width.
