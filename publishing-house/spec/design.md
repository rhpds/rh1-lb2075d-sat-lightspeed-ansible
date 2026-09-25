# Red Hat Satellite: Automated CVE Remediation with Lightspeed and AAP

## Overview

This lab demonstrates how Red Hat Satellite, the on-premises Red Hat Lightspeed Vulnerability service, and Ansible Automation Platform work together to close the loop on CVE remediation without manual intervention. Participants explore the Lightspeed Vulnerability dashboard to review CVEs detected on managed RHEL hosts, wire up a Satellite webhook to Event-Driven Ansible, extend an EDA rulebook to launch a remediation Job Template, and then verify that vulnerable packages are patched fleet-wide — including on hosts that did not fire the triggering event.

## Target Audience

- **Role:** Linux sysadmins, platform engineers, and IT operations teams responsible for RHEL host management and security patching
- **Experience level:** Intermediate
- **What they already know:** Basic RHEL administration, familiarity with Ansible playbooks, general awareness of CVEs and security advisories
- **What they don't know:** How Satellite Lightspeed Vulnerability surfaces CVE data on-premises, how to integrate Satellite with AAP's Event-Driven Ansible, how to build a closed-loop automated remediation pipeline

## Prerequisites

- Basic RHEL system administration (package management with `dnf`, SSH, command-line comfort)
- Basic Ansible familiarity (able to read a playbook; no need to write one from scratch)
- No prior Red Hat Satellite or Ansible Automation Platform experience is required
- Prerequisites cannot be validated automatically; the lab environment is pre-provisioned and self-contained

## Learning Objectives

1. Explore CVE data in the Red Hat Lightspeed Vulnerability dashboard hosted on-premises by Red Hat Satellite
2. Configure a Satellite webhook to send events to AAP's Event-Driven Ansible Event Stream on remote execution job success
3. Automate fleet-wide CVE remediation by wiring an EDA rulebook to launch an AAP Job Template that installs fixes across all affected hosts
4. Verify end-to-end closed-loop remediation by confirming that vulnerable packages are patched on both managed hosts after a single triggering event

## Content Type

Lab (hands-on)

## Products & Technologies

- Red Hat Satellite (with on-premises Red Hat Lightspeed Vulnerability service)
- Red Hat Ansible Automation Platform (AAP) — Controller and Event-Driven Ansible (EDA)
- Red Hat Enterprise Linux (RHEL)

## Module Map

| Module | Title | Duration |
|--------|-------|----------|
| 1 | Introduction | 10 min |
| 2 | Explore Detected CVEs | 20 min |
| 3 | Configure the Satellite Webhook | 20 min |
| 4 | Close the Loop: Automatic Remediation | 20 min |
| 5 | Verify the Closed Loop | 15 min |
| — | **Total hands-on** | **~85 min** |
| — | **Total lab** | **~1.5 hours** |

## Difficulty Level

Intermediate

## Environment

**Learner view:** When the lab starts, four VMs are fully provisioned and running: `satellite.lab` (Satellite server with Lightspeed Vulnerability enabled and CVE data pre-synced), `aap1.lab` (AAP with EDA pre-configured to receive webhooks and pre-placed Ansible playbooks), `rhel1.lab`, and `rhel2.lab` (RHEL hosts registered to Satellite, with deliberately vulnerable packages installed). Participants access the environment through six tabs: Satellite Web UI, AAP Web UI, and terminal sessions for satellite.lab, aap1.lab, rhel1.lab, and rhel2.lab.

**Automation needed:** Yes — setup automation pre-installs and configures Satellite, syncs CVE data (cvemap.xml), registers RHEL hosts, installs vulnerable packages on the RHEL hosts, configures AAP EDA with a minimal rulebook, and pre-places Ansible playbooks on satellite.lab and aap1.lab that participants run during the lab.

## Infrastructure Requirements

- **Cloud provider:** TBD — confirmed in infrastructure phase
- **Cluster type:** TBD — confirmed in infrastructure phase
- **OCP version:** TBD — confirmed in infrastructure phase
- **Topology:** TBD — confirmed in infrastructure phase
- **Sizing:** TBD — confirmed in infrastructure phase
- **Automation approach:** TBD — confirmed in infrastructure phase
- **AI/MaaS:** TBD — confirmed in infrastructure phase
- **External services:** TBD — confirmed in infrastructure phase
- **AAP version:** TBD — confirmed in infrastructure phase
- **Non-GA products:** TBD — confirmed in infrastructure phase

## Assessment Strategy

This is a Zero-Touch Guided lab. Each module ends with a visible, verifiable result:

- Module 2: Participants observe CVE listings with severity ratings in the Lightspeed Vulnerability dashboard
- Module 3: Participants run `verify-webhook.yml` which confirms the Event Stream received the webhook event (`events_received` counter and timestamp visible in playbook output)
- Module 4: All playbooks exit cleanly; EDA Activation log shows the Job Template launched
- Module 5: `rpm -q openssl openssl-libs` on both hosts shows the patched version (`3.5.1-7.el10_1`); the Vulnerability dashboard shows zero Critical/Important CVEs after `insights-client` runs on both hosts

Verification is learner-driven via explicit commands and UI checks documented in each module. There are no automated solve/validate buttons.
