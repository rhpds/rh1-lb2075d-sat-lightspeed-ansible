# Module 03 — Configure the Satellite Webhook

### Brief Overview

This module connects the two halves of the automated remediation pipeline. AAP's EDA side was already configured during lab provisioning — it has a self-hosted rulebook repository, an EDA Project, a Basic Auth credential, an Event Stream, and a running Rulebook Activation. What's missing is the Satellite side: Satellite does not yet know how to call that Event Stream. Participants run two pre-placed Ansible playbooks on satellite.lab to create a custom JSON webhook template in Satellite and then create the webhook itself, which POSTs to AAP's EDA Event Stream every time a remote execution job succeeds on a host. A third playbook verifies the end-to-end delivery by checking the Event Stream counters and the Activation log.

### Audience and Time

- **Personas:** Linux sysadmins, platform engineers comfortable running Ansible playbooks
- **Prerequisites for this module:** Modules 01 and 02 completed; logged into Satellite and AAP Web UIs
- **Estimated duration:** 20 minutes

### Learning Objectives

- Configure a custom JSON webhook template in Satellite using a pre-placed ERB file and Ansible playbook
- Create a Satellite webhook that authenticates to and calls AAP's EDA Event Stream on remote execution job success
- Verify end-to-end webhook delivery by confirming the Event Stream received the event and the Activation log shows the rule fired

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Create the custom JSON webhook template | 6 min |
| 2 | Create the webhook pointing to the EDA Event Stream | 8 min |
| 3 | Verify end-to-end delivery | 6 min |

### Detailed Steps

1. Read the module overview and sequence diagram explaining the Satellite → AAP Gateway → EDA Event Stream → Activation flow
2. On the **satellite.lab terminal** tab, run the webhook template creation playbook:
   ```
   ansible-playbook /root/create-webhook-template.yml
   ```
3. Confirm the playbook exits cleanly; the ERB template is now registered in Satellite
4. On the **satellite.lab terminal** tab, run the webhook creation playbook:
   ```
   ansible-playbook /root/create-webhook.yml
   ```
   (This playbook fetches the EDA Event Stream URL live from AAP's API to avoid stale UUID issues)
5. Confirm the playbook exits cleanly and prints the Basic Auth credentials and Event Stream URL before using them
6. Trigger a test remote execution job to fire the webhook:
   ```
   hammer job-invocation create --job-template "Run Command - Ansible Default" --search-query "name = rhel1.lab" --inputs "command=echo webhook-test"
   ```
7. On the **aap1.lab terminal** tab, run the verification playbook:
   ```
   ansible-playbook /root/verify-webhook.yml
   ```
8. Confirm `events_received` has incremented and `last_event_received_at` shows the current time
9. Click **Next** to proceed to Module 4

### Key Takeaways

- Satellite's built-in webhook templates do not work for the remote execution event type; a custom ERB template is required to emit valid JSON
- The webhook creation playbook fetches the Event Stream URL live from AAP's API on every run — this prevents stale UUID errors if the EDA credential is ever modified
- The current EDA rulebook only logs the event; Module 4 extends it to actually launch a remediation Job Template
- Re-running either playbook is safe — they use idempotent API calls that update in place rather than create duplicates

### Infrastructure Notes

- The custom ERB template file is at `/root/satellite-remote-execution-host-job-json.erb` on satellite.lab (pre-placed during setup)
- The Basic Auth secret file was generated on aap1.lab by the setup automation and saved to the `aap1-user` home directory; the webhook creation playbook reads it over SSH
- The webhook subscribes to the `actions.remote_execution.run_host_job_succeeded` event (displayed with `.event.foreman` suffix in Satellite UI — this is expected)
