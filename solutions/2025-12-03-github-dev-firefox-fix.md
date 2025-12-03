# GitHub.dev Extension Loading Errors on macOS Firefox

**Date:** 2025-12-03
**Platform(s):** macOS, Firefox
**Problem Category:** Web IDE / Networking / Browser Compatibility

## Problem Statement

GitHub.dev failed to load extensions and display workspace files on macOS running Firefox, presenting multiple "NetworkError when attempting to fetch resource" messages and "ENOPRO: No file system provider found" errors. The web IDE was essentially non-functional despite successful authentication and repository access.

## Context

- **Device/System:** macOS
- **Browser:** Firefox (version not specified, but current as of Dec 2025)
- **Platform:** github.dev web-based editor
- **Error Messages:**
  - `NetworkError when attempting to fetch resource` (40+ occurrences in console)
  - `[error] [Netzwerk] https://marketplace.visualstudio.com/_apis/public/gallery/extensionquery - error POST NetworkError`
  - `ENOPRO: Für die Ressource wurde kein Dateisystemanbieter gefunden` (No file system provider found for resource)
  - `Ignoring following additional builtin extensions as there is an error while fetching them from gallery ["github.codespaces","github.github-vscode-theme","github.remotehub","GitHub.vscode-pull-request-github","ms-vscode.anycode"]`
- **Timeline:** Errors began at 2025-12-03 12:58:51.925 CET
- **Repository:** eucalypto/tech-solutions (public GitHub repository)

## Solution

**Two-step fix that resolved the issue completely:**

### Step 1: Disable Ad Blocker
1. Open Firefox
2. Locate your ad blocker extension in the toolbar (e.g., uBlock Origin, Adblock Plus, or equivalent)
3. Click the ad blocker icon
4. Select "Disable on this site" or "Allow ads on github.dev"
5. The extension should show it's now disabled for github.dev

### Step 2: Disable Firefox Enhanced Tracking Protection
1. In Firefox, navigate to github.dev
2. Click the **shield icon** in the address bar (located to the left of the URL)
3. In the popup menu, toggle **Enhanced Tracking Protection** to **OFF** for github.dev
4. You should see a confirmation that protection is disabled for this site
5. Refresh the page (⌘R on macOS)

### Step 3: Verify Resolution
- github.dev should now load all extensions
- The Developer Console (⌘⌥I → Console tab) should show no more "NetworkError" messages
- Workspace files should be accessible in the file tree
- Built-in extensions like GitHub Repositories and GitHub Codespaces should load without errors

**Total resolution time:** Less than 1 minute

## Original Sources

- **Firefox Enhanced Tracking Protection documentation:** https://support.mozilla.org/en-US/kb/enhanced-tracking-protection-firefox-desktop
- **GitHub.dev editor documentation:** https://docs.github.com/en/codespaces/the-githubdev-web-based-editor
- **VS Code web marketplace issues:** Multiple GitHub issues (microsoft/vscode repositories) discussing CORS and network errors in web-based VS Code deployments
- **Firefox privacy extension interactions:** Firefox support forums and community discussions regarding privacy extensions blocking third-party requests

## Key Learnings

### Why This Solution Worked

1. **Ad blocker blocking:** Ad blockers typically intercept all network requests to external domains. The VS Code extension marketplace (`marketplace.visualstudio.com`) is an external third-party service. When disabled, the ad blocker no longer intercepts these requests, allowing github.dev to fetch extension metadata and binaries.

2. **Enhanced Tracking Protection (ETP) blocking:** Firefox's ETP is designed to block third-party tracking requests and cross-origin resource sharing (CORS) requests that appear suspicious. The marketplace API calls are legitimate but cross-origin, so ETP treats them as potential tracking. Disabling ETP for the site allows these requests through.

3. **Cascading failure:** When extensions fail to load due to blocked marketplace requests, the RemoteHub extension (which provides the `vscode-vfs://` filesystem protocol handler) never initializes. Without RemoteHub, VS Code cannot resolve repository file paths, resulting in the "ENOPRO" (No File System Provider) errors.

4. **Error message translation:** The German error messages (`Netzwerk` = Network, `Fenster` = Window) indicate the system language was set to German, but the root cause remains the same across all language configurations.

### Important Caveats

- **Security trade-off:** Disabling Enhanced Tracking Protection reduces privacy protection for that site. Consider this an acceptable trade-off for development tools, as you're actively authenticating with GitHub and trusting the service.

- **Granular whitelisting preferred:** Rather than permanently disabling ETP or ad blockers globally, creating a site-specific exception is the better practice. Most privacy extensions support per-site allowlisting.

- **Private browsing mode incompatibility:** This solution may not work in Firefox Private Browsing mode due to additional storage restrictions. Use a regular window for github.dev.

- **Corporate firewalls:** If on a corporate network, this solution assumes no firewall-level blocking. If errors persist, whitelist these URLs at the firewall level:
  - `https://marketplace.visualstudio.com/*`
  - `https://*.vscode-cdn.net/*`
  - `https://update.code.visualstudio.com/*`

### Why This Wasn't Obvious

The error messages themselves are misleading. The logs show "NetworkError when attempting to fetch resource," which typically indicates genuine network failure, server unavailability, or firewall blocking. In reality, the requests were being silently blocked by the browser's own security mechanisms, not by network infrastructure. This is a common gotcha with browser-based security features—they fail silently with generic error messages.

## Related Issues

- GitHub.dev requires specific browser security settings for VS Code web to function properly
- Similar issues may occur with other browser privacy/security extensions (Privacy Badger, Ghostery, etc.)
- Chrome and Safari with similar privacy extensions may experience identical failures
- github.dev in Firefox Private Browsing mode has additional limitations beyond this issue

## Additional Notes

This solution was discovered through systematic troubleshooting of browser security features as the primary blocking mechanism. The key insight was recognizing that extension marketplace failures cascade into filesystem provider failures, making the initial symptom (ENOPRO errors) a secondary effect of the primary cause (blocked marketplace requests).

---

**Solution verified:** 2025-12-03 13:00 CET
**Status:** ✅ Confirmed working
