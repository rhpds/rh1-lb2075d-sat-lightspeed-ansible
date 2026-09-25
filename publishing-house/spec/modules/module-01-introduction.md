# Module 01 — Introduction

### Brief Overview

This module orients participants to the automated CVE remediation lab. It explains the three-component architecture — Red Hat Satellite (with on-premises Red Hat Lightspeed Vulnerability), Event-Driven Ansible (EDA), and AAP Controller — and describes how they work together to close the remediation loop without manual intervention. The lab environment consists of four pre-provisioned VMs: satellite.lab, aap1.lab, rhel1.lab, and rhel2.lab. The module ends by confirming that participants can log into both the Satellite and AAP Web UIs before the hands-on modules begin.

### Audience and Time

- **Personas:** Linux sysadmins, platform engineers, IT operations teams
- **Prerequisites for this module:** None — this is the first module
- **Estimated duration:** 10 minutes

### Learning Objectives

- Identify the three components (Satellite Lightspeed Vulnerability, EDA, AAP Controller) and their roles in the automated remediation pipeline
- Log into the Satellite Web UI and AAP Web UI using the provided credentials

### Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Lab architecture overview | 3 min |
| 2 | Lab environment description | 2 min |
| 3 | Log into Satellite and AAP Web UIs | 5 min |

### Detailed Steps

1. Read the lab overview describing the three-component pipeline
2. Review the lab environment instances diagram (satellite.lab, aap1.lab, rhel1.lab, rhel2.lab)
3. Click the **Satellite Web UI** tab
4. Log in with credentials: `admin` / `bc31c9a6-9ff0-11ec-9587-00155d1b0702`
5. Use the same credentials to log into the **AAP Web UI** tab
6. Click **Next** to proceed to Module 2

### Key Takeaways

- Red Hat Satellite manages RHEL hosts and, through its on-premises Lightspeed Vulnerability service, maps CVEs to specific hosts and errata
- Event-Driven Ansible (EDA) receives webhook events from Satellite and determines what actions to take
- AAP Controller runs the remediation playbook that installs fixed packages across all affected hosts
- The pipeline works in disconnected environments because all CVE data is synced into Satellite ahead of time

### Infrastructure Notes

- Satellite is pre-installed and content is pre-synchronized — installation is out of scope for this lab
- AAP is pre-configured with an EDA pipeline ready to receive webhooks; Module 3 adds the Satellite-side webhook
- All four VMs must be accessible before proceeding to Module 2
