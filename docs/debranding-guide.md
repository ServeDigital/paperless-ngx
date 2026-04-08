# Paperless-ngx UI De-branding Guide

This document records every change needed to strip paperless-ngx branding from the UI and replace it with neutral or custom branding. Apply these changes to any fresh paperless-ngx fork on the `goldleaf-custom` branch.

All changes are in two locations:
- **Angular frontend** — `src-ui/src/`
- **Django backend templates** — `src/documents/templates/`

---

## 1. Navbar logo — replace leaf SVG with custom logo

**File:** `src-ui/src/app/components/app-frame/app-frame.component.html`

Find the `<a class="navbar-brand ...">` block at the top of the file. It contains a hardcoded paperless-ngx leaf SVG. Replace it:

```html
<!-- BEFORE: paperless-ngx leaf SVG -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 198.43 238.91" width="1em" height="1.5em" fill="currentColor">
  <path d="M194.7,0C164.22,70.94,17.64,79.74,64.55,194.06..." transform="translate(0 0)" />
</svg>

<!-- AFTER: custom logo -->
<img src="assets/goldleaf_logo.png" height="32" alt="Gold Leaf DMS" />
```

The `<a>` block also contains a `<div class="ms-2 ...">` with the app title text — update that text too:

```html
<!-- BEFORE -->
<div class="ms-2 ms-md-3 d-inline-block" ...>
  {{ appTitle }}
</div>

<!-- AFTER -->
<div class="ms-2 ms-md-3 d-inline-block" ...>
  Gold Leaf DMS
</div>
```

Place your logo PNG at `src-ui/src/assets/goldleaf_logo.png`.

---

## 2. User dropdown — remove "Documentation" link

**File:** `src-ui/src/app/components/app-frame/app-frame.component.html`

In the user dropdown (`<div ngbDropdownMenu ...>`), find and remove the Documentation `<a>` and its preceding divider:

```html
<!-- REMOVE this entire block -->
<div class="dropdown-divider"></div>
<a ngbDropdownItem class="nav-link" target="_blank" rel="noopener noreferrer"
  href="https://docs.paperless-ngx.com">
  <i-bs class="me-2" name="question-circle"></i-bs><ng-container i18n>Documentation</ng-container>
</a>
```

---

## 3. Sidebar bottom — remove "Documentation" link

**File:** `src-ui/src/app/components/app-frame/app-frame.component.html`

Search for `tour.outro` — it marks the Documentation `<li>` in the sidebar bottom. Remove the entire `<li>`:

```html
<!-- REMOVE this entire block -->
<li class="nav-item mt-2" tourAnchor="tour.outro">
  <a class="px-3 py-2 text-muted small d-flex align-items-center flex-wrap text-decoration-none"
    target="_blank" rel="noopener noreferrer" href="https://docs.paperless-ngx.com" ngbPopover="Documentation"
    i18n-ngbPopover [disablePopover]="!slimSidebarEnabled" placement="end" container="body"
    triggers="mouseenter:mouseleave" popoverClass="popover-slim">
    <i-bs class="d-flex me-2" name="question-circle"></i-bs><span><ng-container i18n>Documentation</ng-container></span>
  </a>
</li>
```

---

## 4. Sidebar version footer — remove GitHub link and update-check popup

**File:** `src-ui/src/app/components/app-frame/app-frame.component.html`

The version string at the bottom of the sidebar is wrapped in an `<a>` linking to the paperless-ngx GitHub repo, and is followed by an update-checking section that contains "Paperless-ngx" branding. Replace the entire `<li>` with a plain version text:

```html
<!-- BEFORE: entire block with GitHub link + update-check popup -->
<li class="nav-item" [class.visually-hidden]="slimSidebarEnabled">
  <div class="px-3 py-0 text-muted small d-flex align-items-center flex-wrap">
    <div class="me-3">
      <a class="text-muted text-decoration-none" target="_blank" rel="noopener noreferrer"
        href="https://github.com/paperless-ngx/paperless-ngx" ngbPopover="GitHub" i18n-ngbPopover
        [disablePopover]="!slimSidebarEnabled" placement="end" container="body"
        triggers="mouseenter:mouseleave" popoverClass="popover-slim">
        {{ versionString }}
      </a>
    </div>
    @if (!settingsService.updateCheckingIsSet || appRemoteVersion) {
      <div class="version-check">
        <ng-template #updateAvailablePopContent>
          <span class="small">Paperless-ngx {{ appRemoteVersion.version }} <ng-container i18n>is
              available.</ng-container>...</span>
        </ng-template>
        <ng-template #updateCheckingNotEnabledPopContent>
          <p class="small mb-2">
            <ng-container i18n>Paperless-ngx can automatically check for updates</ng-container>
          </p>
          ...
        </ng-template>
        ...
      </div>
    }
  </div>
</li>

<!-- AFTER: plain version text, no links -->
<li class="nav-item" [class.visually-hidden]="slimSidebarEnabled">
  <div class="px-3 py-0 text-muted small">
    {{ versionString }}
  </div>
</li>
```

---

## 5. Settings page — remove "Open Django Admin" button

**File:** `src-ui/src/app/components/admin/settings/settings.component.html`

In the `<pngx-page-header>` block, remove the Django Admin link:

```html
<!-- REMOVE this entire element -->
<a class="btn btn-sm btn-primary" href="admin/" target="_blank">
  <ng-container i18n>Open Django Admin</ng-container>
  <i-bs class="ms-2" name="arrow-up-right"></i-bs>
</a>
```

---

## 6. Django loading screen — remove paperless-ngx references

**File:** `src/documents/templates/index.html`

This is the HTML shell served before Angular loads. Find the `app-loader` div and update it:

```html
<!-- BEFORE -->
{% include "paperless-ngx/snippets/svg_logo.html" with extra_attrs="class='logo mb-2' height='6em'" %}
<h6 class="m-auto">{% translate "Paperless-ngx is loading..." %}</h6>
<p class="warning m-auto mt-3 small fade hide">{% translate "Still here?! Hmm, something might be wrong." %} <a href="https://docs.paperless-ngx.com">{% translate "Here's a link to the docs." %}</a></p>

<!-- AFTER -->
<h6 class="m-auto">{% translate "Gold Leaf DMS is loading..." %}</h6>
<p class="warning m-auto mt-3 small fade hide">{% translate "Still here?! Hmm, something might be wrong. Please try refreshing the page." %}</p>
```

Changes:
- Remove the `{% include %}` tag for the paperless SVG logo
- Change "Paperless-ngx is loading..." to your app name
- Remove the `docs.paperless-ngx.com` fallback link

---

## 7. Register the `file-earmark-pdf` icon (if adding PDF Tools nav item)

**File:** `src-ui/src/main.ts`

If you add a "PDF Tools" sidebar link using `<i-bs name="file-earmark-pdf">`, the icon must be registered. The icons list only includes icons that are explicitly imported:

```typescript
// In the import block from 'ngx-bootstrap-icons':
import {
  ...
  fileEarmarkPdf,     // ← add this
  fileEarmarkRichtext,
  ...
} from 'ngx-bootstrap-icons'

// In the icons object:
const icons = {
  ...
  fileEarmarkPdf,     // ← add this
  fileEarmarkRichtext,
  ...
}
```

Without this, `<i-bs name="file-earmark-pdf">` renders as an empty element with no SVG.

---

## Summary of files changed

| File | What changed |
|---|---|
| `src-ui/src/app/components/app-frame/app-frame.component.html` | Navbar logo, user dropdown, sidebar bottom, version footer |
| `src-ui/src/app/components/admin/settings/settings.component.html` | Removed Django Admin button |
| `src-ui/src/main.ts` | Registered `fileEarmarkPdf` icon |
| `src/documents/templates/index.html` | Loading screen text and logo |

---

## What is intentionally NOT changed

These files use `paperless-ngx` as Django template directory names — this is internal Django template inheritance syntax, not user-visible text. Do not change them:

```
src/documents/templates/account/login.html        → {% extends "paperless-ngx/base.html" %}
src/documents/templates/account/signup.html       → {% extends "paperless-ngx/base.html" %}
src/documents/templates/paperless-ngx/base.html   → the actual base template (edit content here)
```

The directory name `paperless-ngx/` inside `templates/` is referenced throughout Django and would require renaming dozens of files to change. The user-visible content is in `base.html` itself — that is where branding edits go.

---

## Verification

After applying changes, verify in the browser at `http://localhost:4200`:

- [ ] Navbar top-left: custom logo, no leaf SVG
- [ ] User dropdown (click avatar): My Profile, Settings, Logout — no Documentation link
- [ ] Sidebar bottom: version string is plain text, not a GitHub link
- [ ] Sidebar bottom: no Documentation link above version string
- [ ] `/settings` page: no "Open Django Admin" button in the header
- [ ] Page source (`Ctrl+U` at `:8000`): loading message says your app name, not "Paperless-ngx is loading..."
- [ ] No `docs.paperless-ngx.com` or `github.com/paperless-ngx` links anywhere (verify with browser DevTools → Elements search)
