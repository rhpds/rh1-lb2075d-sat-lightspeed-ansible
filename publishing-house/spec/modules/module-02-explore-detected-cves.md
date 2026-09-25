# Module 02 — Explore Detected CVEs

### Brief Overview

Before wiring up any automation, participants explore the CVE data Satellite has already collected. The two managed RHEL hosts (rhel1.lab and rhel2.lab) were seeded with deliberately vulnerable packages during lab setup. Both hosts have reported their package inventory back to Satellite via `insights-client`, and Satellite has cross-referenced that data against Red Hat's published CVE information to produce an on-premises vulnerability dashboard. No data leaves the lab network during this process. This module walks participants through the Lightspeed Vulnerability dashboard — filtering by severity, reading per-host impact counts, and drilling into individual CVE detail pages to understand what errata are available to fix each vulnerability.

### Audience and Time

- **Personas:** Linux sysadmins, platform engineers
- **Prerequisites for this module:** Module 01 completed; logged into Satellite Web UI
- **Estimated duration:** 20 minutes

### Learning Objectives

- Navigate to the Red Hat Lightspeed Vulnerability dashboard within the Satellite Web UI
- Interpret CVE severity ratings (Critical/Important/Moderate/Low), CVSS scores, and systems-affected counts
- Identify which CVEs have remediating errata available in Satellite vs. those that are not yet actionable

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Open the Vulnerability dashboard | 5 min |
| 2 | Review and filter detected CVEs | 8 min |
| 3 | Drill into one CVE's detail page | 7 min |

### Detailed Steps

1. In the Satellite Web UI, navigate to **Red Hat Lightspeed → Vulnerability**
2. Observe that the dashboard is served entirely on-premises from CVE data pre-synced into Satellite
3. Review the list of CVEs: note severity ratings, CVSS scores, and Systems Affected counts
4. Sort by **Severity** to see Critical and Important CVEs first
5. Note that not all CVEs have an actionable erratum — only those with a synced fix appear as remediable
6. Click on any CVE affecting 2 systems to open its detail page
7. Review the full CVE description, CVSS score, affected hosts list, and available erratum
8. Click **Next** to proceed to Module 3

### Key Takeaways

- The Lightspeed Vulnerability dashboard is served by Satellite itself, not the hosted `console.redhat.com` — it is fully on-premises
- A CVE becomes actionable only once Satellite has synced the erratum (patch) that resolves it
- Systems Affected counts link directly to the specific managed hosts impacted by each CVE
- CVE severity and CVSS scores help prioritize which vulnerabilities to remediate first

### Infrastructure Notes

- The `insights-client` on each RHEL host reports the installed package list to Satellite; Satellite performs CVE matching locally using its synced CVE data
- The seeded vulnerable packages are documented in the lab setup script (`setup-automation/setup-satellite.sh`)
