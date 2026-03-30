# Cursor Prompt: Adaptive Security – BDR Booking Tool

## Project Overview

Build a single-page web application called **"Adaptive Booking"** — a mobile-optimized prospect booking tool for Adaptive Security BDRs. The app collects prospect information, routes to the correct ChiliPiper meeting link based on company size, and pre-fills the booking form automatically.

---

## Tech Stack

- **Framework**: Vanilla HTML/CSS/JS (single `index.html` file — no build step required)
- **ChiliPiper**: Loaded via their CDN script tag (`https://js.chilipiper.com/marketing.js`)
- **Fonts**: Google Fonts (your choice — see Design section)
- **No external dependencies** beyond ChiliPiper and Google Fonts

---

## Application Flow

### Step 1 — Prospect Info Form
The user sees a clean form with the following fields:

| Field | Type | Required |
|---|---|---|
| First Name | text input | yes |
| Last Name | text input | yes |
| Email | email input | yes |
| Company Size | segmented selector (not a dropdown) | yes |
| Additional Guest Emails | dynamic tag-input (add/remove emails) | no |

**Company Size options** (displayed as pill/toggle buttons, not a dropdown):
- `< 500`
- `500 – 4,999`
- `5,000+`

**Guest email UX**: User types an email and presses Enter or comma to add it as a removable tag. Show each guest as a chip with an ✕ to remove.

Validate all required fields before proceeding. Show inline validation errors.

### Step 2 — ChiliPiper Booking Widget
After the form is submitted, the prospect info form slides out and the ChiliPiper calendar interface slides in — rendered inline in the page (not a popup or redirect).

**Routing logic** based on company size:

```
< 500        → https://adaptivesecurity.chilipiper.com/round-robin/outbound-default-meeting-45-mm
500 – 4,999  → https://adaptivesecurity.chilipiper.com/round-robin/outbound-default-meeting-45-ent
5,000+       → https://adaptivesecurity.chilipiper.com/round-robin/strat-default-meeting-47
```

**ChiliPiper integration**:

Load ChiliPiper via script tag:
```html
<script src="https://js.chilipiper.com/marketing.js" type="text/javascript" async></script>
```

Trigger the booking widget using:
```javascript
ChiliPiper.submit("adaptivesecurity", ROUTE_NAME, {
  title: "Book a Meeting",
  map: true,
  lead: {
    FirstName: firstName,
    LastName: lastName,
    Email: email,
    // additional guests handled separately
  },
  onSuccess: function() {
    showConfirmationScreen();
  }
});
```

Where `ROUTE_NAME` is extracted from the URL path (e.g., `"outbound-default-meeting-45-mm"`).

**Guest emails**: After ChiliPiper renders, attempt to auto-populate the guest/attendee field. ChiliPiper exposes a DOM element for additional guests — query for it and inject the guest emails as comma-separated values once the iframe/widget has loaded. Use a `MutationObserver` to detect when the ChiliPiper widget DOM is ready, then inject.

### Step 3 — Confirmation Screen
After booking is confirmed (ChiliPiper `onSuccess` callback), show a clean confirmation screen:
- "Meeting booked! ✓"
- Prospect name and email
- A "Book Another" button that resets the entire form

---

## Design Direction

**Aesthetic**: Dark, precise, modern — like a premium SaaS internal tool. Think Bloomberg Terminal meets a fintech app. Sharp edges, high contrast, no rounded-corner softness. Make it feel like a weapon, not a toy.

**Color palette**:
- Background: `#0a0a0f` (near black)
- Surface/card: `#111118`
- Border: `#1e1e2e`
- Primary accent: `#00e5ff` (electric cyan)
- Text primary: `#f0f0f5`
- Text muted: `#6b6b80`
- Error: `#ff4d6d`
- Success: `#00c9a7`

**Typography**:
- Display/labels: `"DM Mono"` from Google Fonts — monospaced, technical, authoritative
- Body: `"Inter"` from Google Fonts — clean and legible
- Use tight letter-spacing on headings, uppercase labels

**Layout**:
- Mobile-first, max-width `480px`, centered on desktop
- Full-height (`100dvh`) layout
- The form card should have a subtle `box-shadow` glow using the accent color at low opacity
- Header: small Adaptive Security wordmark (text-only, styled) + "BDR Booking Tool" label

**Animations**:
- Form → Widget transition: horizontal slide (form slides left out, widget slides in from right)
- Field focus: subtle cyan underline animation
- Button hover: background fill sweep left-to-right
- Guest tag add/remove: scale + fade
- Confirmation screen: staggered fade-up on each element

**Segmented company size selector**:
- Three pill buttons in a row, full width
- Inactive: dark surface with muted border
- Active: filled with accent color (`#00e5ff`), black text, no border

**Input fields**:
- Flat, no border-radius, bottom-border-only style
- Label floats above on focus/fill
- Cyan bottom border on focus

**Submit button**:
- Full width
- Background: `#00e5ff`
- Text: `#0a0a0f` (black), uppercase, monospaced font, bold
- Hover: slight glow effect

---

## File Structure

Output a single `index.html` file. Embed all CSS in a `<style>` tag and all JS in a `<script>` tag at the bottom of `<body>`.

---

## Additional Requirements

1. **No page reloads** — the entire experience is SPA-style within one page
2. **ChiliPiper container**: Create a `<div id="chilipiper-wrapper">` that ChiliPiper renders into. Style it to take full available height after the header.
3. **Back button**: On the ChiliPiper step, show a small "← Back" link in the top left that returns to the form (in case rep needs to correct info)
4. **Error handling**: If ChiliPiper fails to load or the script times out, show a fallback message with the direct booking URL so the rep can still send it manually
5. **Clipboard copy**: On the fallback error state, include a "Copy Link" button
6. **Touch targets**: All interactive elements must be at least 44px tall for mobile usability
7. **No scrollbar on body** during the booking widget step — let ChiliPiper's iframe handle its own scroll
8. **Guest tag input**: Support paste of comma-separated emails (e.g., pasting `a@co.com, b@co.com` adds two tags at once)

---

## ChiliPiper Implementation Notes

ChiliPiper's JS SDK fires against a specific tenant (`adaptivesecurity`) and route slug. The three route slugs to use are:

- `outbound-default-meeting-45-mm` (< 500)
- `outbound-default-meeting-45-ent` (500–4,999)
- `strat-default-meeting-47` (5,000+)

The `ChiliPiper.submit()` call should be made **after** the user clicks "Find a Time" and the form validates successfully. The widget renders inside the designated container div — do not use the popup/modal mode.

For guest email injection: ChiliPiper's widget renders an `<iframe>`. Direct DOM manipulation inside the iframe is cross-origin blocked. Instead, pass guests via the `lead` object if ChiliPiper supports a `GuestEmails` or `cc` field, OR display the guest emails prominently on the confirmation screen with a "Copy guest emails" button so the rep can paste them into the calendar invite manually. Add a note in the UI: *"Add guests manually after booking — emails copied below."*

---

## Deliverable

One complete, working `index.html` file ready to open in a browser or deploy to any static host (Netlify, Vercel, GitHub Pages). No npm, no build tools required.
