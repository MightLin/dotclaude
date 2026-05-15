    # Design Brief: shifttable-redesign
    Date: 2026-05-11
    Mode: greenfield

    ## Project Scope
    - Product: ShiftTable clinic staff scheduling system
    - Primary users: clinic managers and administrative schedulers
    - Secondary users: staff members and doctors who need schedule visibility, leave status, or calendar sync
    - Platform: web first, with responsive support for tablet and mobile
    - Design direction: modern SaaS with a gentle healthcare feel; clean, trustworthy, efficient, data-dense without feeling heavy

    ## Product Context
    - ShiftTable is an Angular SPA backed by Firebase.
    - Core scheduling data and workflows live around the shift table, staff sidebar, AI scheduling dialog, rule checking, leave settings, calendar view, print view, and Google Calendar sync.
    - The UI should be designed as an operational workspace, not a marketing site or landing page.
    - The first screen should immediately show the scheduling workflow.

    ## Core Workflows
    - View and edit monthly or weekly clinic schedules in a grid.
    - Scan staffing coverage by day and shift type.
    - Assign, move, or adjust staff shifts.
    - Review AI-generated schedules and controlled AI adjustment suggestions.
    - Inspect scheduling rule errors and warnings.
    - See leave, preference, role, and availability constraints while scheduling.
    - Sync completed schedules to Google Calendar.
    - Print or export a clean schedule view.

    ## Required UI Surfaces
    - Schedule workspace with month/week navigation and grid density controls
    - Staff sidebar with role, availability, leave, and shift count summary
    - Shift cells for early, afternoon, and evening shifts
    - AI scheduling control area with generate, adjust, preview, and apply states
    - Rule check panel with error, warning, and valid states
    - Leave/preference visibility integrated into scheduling context
    - Google Calendar sync status and action feedback
    - Print/export-friendly schedule view
    - Loading, empty, error, and partial-sync states

    ## Visual Direction
    - Tone: professional, calm, modern, operational
    - Avoid oversized hero sections, decorative cards, marketing-style composition, and purely illustrative layouts.
    - Use restrained medical teal or blue-green accents, balanced with neutral surfaces and a small amount of indigo, slate, amber, green, and red for status.
    - Shift colors should be easy to scan without turning the full interface into a single-hue palette.
    - Prioritize strong hierarchy, compact controls, clear spacing, and accessible contrast.
    - Keep cards to repeated items, dialogs, and genuinely framed panels. Do not nest cards inside cards.

    ## Interaction Expectations
    - Scheduling should support fast scanning, quick reassignment, and low-friction corrections.
    - AI suggestions should feel inspectable and reversible; automation must not feel like an opaque one-click takeover.
    - Rule violations should be visible near the affected schedule context and also summarized in a dedicated panel.
    - Users should always understand whether a problem is blocking, warning-only, resolved, or waiting for sync.
    - Mobile views should preserve schedule context through segmented views, horizontal navigation, or focused day/employee drill-ins.

    ## Technical Constraints
    - Angular 20 standalone components
    - PrimeNG 19+ with Aura theme
    - Light mode only
    - Tailwind CSS utilities for layout
    - No PrimeFlex classes
    - No Angular Material
    - New buttons must use the `p-button` component, not the `pButton` directive
    - Form controls should align with PrimeNG components such as input, select, dialog, message, badge, and related controls
    - Design should be practical to implement in the existing Angular + Firebase codebase

    ## AI Design Tool Prompt
    Design a greenfield UI redesign for ShiftTable, a clinic staff scheduling web app. The first screen should be the actual scheduling workspace, not a marketing landing page. Create a modern SaaS-style operational interface with a gentle healthcare feel: clean, trustworthy, efficient, and suitable for repeated daily use.

    The app is used mainly by clinic managers and administrative schedulers. The core workspace should include a monthly or weekly schedule grid, staff list or sidebar, shift type indicators for early, afternoon, and evening shifts, AI scheduling controls, rule validation results, leave and preference visibility, and Google Calendar sync status.

    Use a light theme inspired by PrimeNG Aura. The layout should feel compatible with Angular, PrimeNG, and Tailwind implementation. Avoid Angular Material patterns, PrimeFlex-like class assumptions, and decorative marketing sections. Use compact toolbars, clear icon buttons, segmented controls, tabs, dialogs, badges, status messages, and dense but readable tables or grids.

    Prioritize fast scanning of staffing coverage by day and shift, clear difference between errors, warnings, and valid states, easy drag/drop or reassignment affordances, visible AI suggestions without making automation feel uncontrolled, responsive behavior for tablet and mobile, and print/export-friendly structure.

    The visual tone should be professional, calm, and modern, with restrained medical teal and blue accents, neutral surfaces, strong text hierarchy, and accessible contrast.
