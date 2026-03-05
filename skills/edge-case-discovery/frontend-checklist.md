# Frontend Edge Case Checklist

Additional edge cases specific to user interfaces and client-side applications.

## Rendering States

- Loading state (data not yet available)
- Empty state (data loaded but no items exist)
- Error state (data failed to load)
- Partial data (some fields present, others still loading or failed)
- Stale data (cache shows old data while fresh data loads)
- Skeleton/placeholder content before hydration

## User Input

- Rapid repeated clicks (double-submit of forms, double navigation)
- Paste into input fields (bypasses keystroke validation)
- Autofill by browser or password manager (may not trigger change events)
- Input while previous async operation is still in flight
- Copy-pasting text with hidden formatting or zero-width characters
- Extremely long text in free-form fields (layout overflow)

## Navigation and State

- Browser back/forward through multi-step flow (wizard, checkout)
- Deep linking directly to a step that requires prior state
- Page refresh mid-operation (form data lost, state reset)
- Multiple tabs with the same application (shared state conflicts)
- Bookmark or share a URL that contains ephemeral state

## Network Conditions

- Offline: what does the user see? Can they do anything useful?
- Slow connection: do operations appear hung? Is there feedback?
- Connection restored: does stale data refresh automatically?
- Request succeeds but response is lost (operation completed server-side but client shows error)

## Display and Layout

- Text content longer than expected (names, descriptions, translations)
- Text content shorter than expected (single character, empty after trim)
- Screen sizes: mobile, tablet, desktop, ultra-wide
- Zoom levels: 50% through 200%
- Right-to-left (RTL) text direction
- High contrast and dark mode

## Accessibility

- Keyboard-only navigation (all interactive elements reachable via Tab)
- Screen reader announces state changes (loading, errors, success)
- Focus management after modal open/close, route change, dynamic content
- Color is not the sole indicator of state (add icons, text, or patterns)
- Touch targets are large enough on mobile (44x44px minimum)
