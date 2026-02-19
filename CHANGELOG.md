
nobot releases and changes
================================================================================


0.4.0 (2026-02-19)
----------------------------------------

-   BREAKING: Renamed two modules. Please update your config to use the new names:
    -   From *30-unexpected-anonymous.conf* to *30-anonymous.conf*
    -   From *35-unexpected-anonymous-essentially.conf* to *35-anonymous-essentially.conf*
-   Switched from code 451 to code 402 for modules 20, 30, and 35
-   Consolidated module 20 from two rules into one
-   Added browser-supports-HTTP/2 conditions to the skip for module 42
-   Added NOBOT env var to module 10
-   Added documentation to env var comments in 01-base.conf
-   Added more patterns, conditions, and rules



0.3.1 (2026-02-16)
----------------------------------------

-   Disabled bad rule for BingPreview bot in 80-bot-impersonators
-   Added more patterns, conditions, and rules



0.3.0 (2026-02-15)
----------------------------------------

-   Removed dependency on Apache TLS configuration from the unexpected-tls-version module:
    -   The module is now safe to include regardless of TLS configuration
-   Tightened regular expressions for Samsung Internet 10 and newer
-   Added more patterns, conditions, and rules



0.2.1 (2026-02-13)
----------------------------------------

-   Moved Safari to second place, above Edge and Firefox, in browser identification
    -   Why: Safari has more market share than Edge and Firefox combined in all continents
-   Tightened regular expressions for CHROME_0_TO_... ranges
-   Added documentation to the file header of the base module
-   Added more patterns, conditions, rules, and skips
-   Added missing resource to the README



0.2.0 (2026-02-12)
----------------------------------------

-   Added early exits to skip non-matching browser sections in 01-base.conf
-   Fixed underskipping for Chrome, Edge, and Firefox sections
-   Sample gains in regex evaluations needed for current browsers:
    -   Chrome 144: from 20 to 14
    -   Edge Desktop 144: from 33 to 10
    -   Firefox 147: from 48 to 17
    -   Safari 26: from 61 to 20
    -   Samsung Internet 29: from 66 to 10


0.1.1 (2026-02-12)
----------------------------------------

-   Improved README, added two new examples to it
-   Added patterns, conditions, and rules
-   Removed unused variables


0.1.0 (2026-02-11)
----------------------------------------

-   Initial public release of nobot
-   Made nobot repository public on GitHub
