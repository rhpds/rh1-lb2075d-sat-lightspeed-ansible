# Module 05 — Verify the Closed Loop

### Brief Overview

Module 4 wired everything together. A successful Satellite remote execution job now triggers a Satellite webhook, which EDA routes to a running rulebook, which launches an AAP Job Template, which finds and remediates all Critical and Important CVEs fleet-wide. This module proves it. Participants check the current (deliberately old) OpenSSL package versions on both rhel1.lab and rhel2.lab, trigger a remote execution job on rhel1.lab, watch AAP launch and complete the remediation Job Template, and then confirm that patched OpenSSL versions appear on both hosts — including rhel2.lab, which never fired a webhook of its own. The module closes by running `insights-client` on both hosts to sync the updated package inventory back to Satellite's Vulnerability dashboard.

### Audience and Time

- **Personas:** Linux sysadmins, platform engineers
- **Prerequisites for this module:** Module 04 completed; EDA rulebook wired and Activation running
- **Estimated duration:** 15 minutes

### Learning Objectives

- Verify pre-remediation package versions on both managed hosts
- Trigger the closed-loop pipeline by running a remote execution job on one host and observing the fleet-wide remediation
- Confirm post-remediation package versions on both hosts and sync the updated inventory to Satellite's Vulnerability dashboard

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Check current vulnerable package versions | 3 min |
| 2 | Trigger the remote execution job | 3 min |
| 3 | Watch the Job Template run | 5 min |
| 4 | Confirm remediation and update Satellite inventory | 4 min |

### Detailed Steps

1. On the **rhel1.lab terminal** tab, check current (vulnerable) package versions:
   ```
   rpm -q openssl openssl-libs gnutls tar
   ```
2. Note the old OpenSSL versions (`3.5.1-4.el10_1`) and that `gnutls` and `tar` on rhel1.lab are also old
3. On the **rhel2.lab terminal** tab, run the same command and note that rhel2.lab has the same old OpenSSL versions but current `gnutls` and `tar`
4. On the **satellite.lab terminal** tab, trigger a remote execution job on rhel1.lab:
   ```
   hammer job-invocation create \
     --job-template "Run Command - Ansible Default" \
     --search-query "name = rhel1.lab" \
     --inputs "command=echo trigger-remediation"
   ```
5. On the **aap1.lab terminal** tab, poll for the Job Template to launch and complete:
   ```
   export CTRL_API="https://localhost/api/controller/v2"
   export CTRL_AUTH="admin:bc31c9a6-9ff0-11ec-9587-00155d1b0702"
   for i in $(seq 1 30); do
     JOB=$(curl -sk -u "$CTRL_AUTH" "$CTRL_API/jobs/?job_template__name=Vulnerability%20Package%20Finder%20and%20Remediator&order_by=-id&page_size=1" \
       | python3 -c "import sys,json; r=json.load(sys.stdin)['results']; print(json.dumps(r[0]) if r else '')")
     if [ -n "$JOB" ]; then
       STATUS=$(echo "$JOB" | python3 -c "import sys,json; print(json.load(sys.stdin)['status'])")
       echo "status=$STATUS"
       [ "$STATUS" = "successful" ] && break
       [ "$STATUS" = "failed" ] && { echo "$JOB" | python3 -m json.tool; break; }
     fi
     sleep 3
   done
   ```
6. Confirm `status=successful`; review the stdout to see which packages were installed on each host
7. On the **rhel1.lab terminal** tab, verify patched OpenSSL:
   ```
   rpm -q openssl openssl-libs gnutls tar
   ```
   (expect `openssl-3.5.1-7.el10_1`, `gnutls` and `tar` unchanged — their errata are only Moderate)
8. On the **rhel2.lab terminal** tab, run the same command and confirm rhel2.lab also has patched OpenSSL
9. Run `insights-client` on **rhel1.lab**:
   ```
   insights-client
   ```
10. Run `insights-client` on **rhel2.lab**:
    ```
    insights-client
    ```
11. In the **Satellite Web UI**, navigate to **Red Hat Lightspeed → Vulnerability** and confirm the Important-severity OpenSSL CVE is no longer listed

### Key Takeaways

- A webhook from a single host (rhel1.lab) triggers fleet-wide remediation — rhel2.lab is patched even though it never fired an event
- Only Critical and Important CVEs are auto-remediated; `gnutls` and `tar` on rhel1.lab are not upgraded because their errata are only Moderate
- `rpm -q` reflects the new versions immediately after the AAP job finishes; the Satellite Vulnerability dashboard lags until `insights-client` uploads an updated package inventory
- Running `insights-client` on both hosts is required to clear the CVE from the dashboard — Satellite only removes the CVE once every affected host has reported a clean package list

### Infrastructure Notes

- The `PLAY RECAP` in the Job Template stdout shows one line per affected host: `changed` means packages were installed, `skipped` means the host was already up to date
- If the Job Template fails with `Permission denied (publickey)`, the Machine credential is missing from the Job Template (Module 4 Step 3)
- If `insights-client` exits with an error, wait and re-run before reloading the Vulnerability dashboard
