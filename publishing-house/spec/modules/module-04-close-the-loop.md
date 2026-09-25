# Module 04 — Close the Loop: Automatic Remediation

### Brief Overview

Module 3 wired the EDA side so that Satellite's webhook reaches a running rulebook — but right now that rulebook only logs the event. This module extends it to perform real remediation. Participants review the `vulnerability_remediation.py` script and `find_and_remediate.yml` playbook (pre-placed on aap1.lab), create a custom Satellite API credential type and instance in AAP Controller, set up a Controller Project and Job Template that runs the remediation playbook, and finally wire the EDA rulebook to automatically launch that Job Template whenever a remote execution job succeeds on any managed host. When triggered, the playbook queries Satellite's Lightspeed Vulnerability API and Katello errata API to find every host affected by a Critical or Important CVE, then installs exactly the packages each host needs.

### Audience and Time

- **Personas:** Linux sysadmins, platform engineers, AAP operators
- **Prerequisites for this module:** Module 03 completed; webhook verified end-to-end
- **Estimated duration:** 20 minutes

### Learning Objectives

- Review the vulnerability_remediation.py script and understand how it queries Satellite's on-premises Lightspeed Vulnerability and Katello errata APIs
- Create a custom Satellite API credential type and credential in AAP Controller using a pre-placed Ansible playbook
- Set up an AAP Controller Project and Job Template that runs the fleet-wide CVE remediation playbook
- Wire the EDA rulebook to launch the Job Template automatically on webhook receipt

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Review the remediation files | 5 min |
| 2 | Create the Satellite API credential | 5 min |
| 3 | Create the Controller Project and Job Template | 5 min |
| 4 | Wire the rulebook to launch the Job Template | 5 min |

### Detailed Steps

1. On the **aap1.lab terminal** tab, list the vulnerability remediation files:
   ```
   sudo -u aap1-user bash -c 'ls -la ~/vulnerability-remediation'
   ```
2. Review `find_and_remediate.yml` to understand how it uses `vulnerability_remediation.py` per host:
   ```
   sudo -u aap1-user bash -c 'cat ~/vulnerability-remediation/find_and_remediate.yml'
   ```
3. Create the Satellite API credential type and credential in Controller:
   ```
   ansible-playbook /root/create-satellite-credential.yml
   ```
4. Confirm both tasks report `ok` or `changed` (re-running is safe due to `state: present`)
5. Create the Controller Project and Job Template:
   ```
   ansible-playbook /root/create-controller-project.yml
   ```
6. Wait for the Project sync to complete (the playbook waits automatically)
7. Wire the EDA rulebook to launch the Job Template:
   ```
   ansible-playbook /root/wire-rulebook.yml
   ```
8. Confirm the playbook: creates an AAP Controller credential in EDA, extends the rulebook with a second action, resyncs the EDA Project, and recreates the Activation
9. Click **Next** to proceed to Module 5

### Key Takeaways

- `vulnerability_remediation.py` queries Satellite's on-premises APIs only — the Lightspeed Vulnerability endpoint (path contains `insights_cloud`) is served by Satellite itself, not `console.redhat.com`
- The remediation is fleet-wide: a webhook from a single host triggers remediation across *every* host affected by a Critical or Important CVE, not just the host that fired the event
- A custom Execution Environment (pre-built during lab setup) is required because `vulnerability_remediation.py` uses a `uv` shebang that AAP's stock EEs do not ship
- `ask_variables_on_launch: true` on the Job Template is required to allow the EDA rulebook to pass `host_name` as an extra var at launch time

### Infrastructure Notes

- The remediation script and playbook are self-hosted as a git repo at `~/vulnerability-remediation` on aap1.lab, mirroring the EDA webhook repo pattern from Module 3
- The Satellite API credential uses `!unsafe` in its injectors block — this is required, not optional, to prevent Ansible from trying to resolve Controller's template variables at credential-type creation time
- Only CVEs rated Critical or Important are auto-remediated (`DEFAULT_SEVERITIES = ["Critical", "Important"]` in the script); Moderate and Low CVEs appear in the dashboard but are intentionally skipped
