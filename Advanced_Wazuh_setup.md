# Advanced Wazuh Security Monitoring & Threat Detection

This document builds upon the foundational Wazuh Server and Windows Agent deployment, extending the capabilities of the Security Operations Center (SOC) to include multi-OS monitoring, File Integrity Monitoring (FIM), and advanced telemetry via Sysmon and Windows Defender.

## 📌 Objectives
1. Deploy a Linux Endpoint Agent.
2. Configure File Integrity Monitoring (FIM) across Windows and Linux.
3. Integrate native Windows Defender logs into Wazuh.
4. Install Sysmon and forward deep system telemetry to Wazuh.
5. Create custom detection rules to identify specific malicious activities.

---

## 🐧 Phase 1: Linux Endpoint Deployment

**Steps Taken:**
- [x] Prepared a Linux VM (Kali Linux) on the Bridged Network.
- [x] Generated the deployment script via Wazuh Dashboard.
- [x] Installed and started the `wazuh-agent` service.
- [x] Verified real-time log collection in the dashboard.

**Screenshots Captured:**
- `linux_agent_active.png`: Captured from the dashboard showing the Windows 11 and Kali Linux agents successfully connected and active.

---

## 📁 Phase 2: File Integrity Monitoring (FIM)

File Integrity Monitoring is a core Wazuh module that detects changes (creations, modifications, deletions) to files or directories. While Wazuh monitors some critical system files by default, we will configure it to monitor specific test directories in real-time on both our Windows and Kali endpoints to validate that our alerts are functioning.

**Steps to Complete:**
- [x] Create a test directory on the Windows endpoint.
- [x] Configure `ossec.conf` on Windows to monitor the new directory in real-time.
- [x] Create a test directory on the Kali Linux endpoint.
- [x] Configure `ossec.conf` on Kali to monitor the new directory in real-time.
- [x] Trigger an alert by creating/modifying a file and capture the dashboard alert.

**Screenshots Captured:**
- `windows_fim_config.png`: Documented the real-time `<directories>` addition in `ossec.conf`.
- `linux_fim_config.png`: Documented the Kali terminal configuration for FIM.
- `fim_alert_dashboard.png`: Captured the dashboard successfully detecting creations and modifications on both the Windows and Kali endpoints.

---

## 🛡️ Phase 3: Windows Defender Integration

To achieve comprehensive monitoring, Wazuh must be able to ingest native Windows Defender event logs. We will configure the Windows agent to read from the Defender Event Channel, allowing Wazuh to generate alerts whenever Defender detects malware or takes action.

**Steps to Complete:**
- [x] Add the Windows Defender Event Channel to `ossec.conf` on the Windows endpoint.
- [x] Restart the Wazuh agent to apply changes.
- [x] Trigger a harmless Defender alert using the EICAR test file.
- [x] Capture the resulting malware alert in the Wazuh Dashboard.

**Screenshots Captured:**
- `defender_config.png`: Captured the `<localfile>` block addition in the agent configuration.
- `defender_alert_dashboard.png`: Captured the "Severe" severity log for `virus_test.txt` ingested natively from Windows Defender into the Wazuh Discover tab.

---

## 👁️ Phase 4: Sysmon Installation & Integration

While default Windows event logs are helpful, they lack deep visibility into process creations, network connections, and file modifications. System Monitor (Sysmon) by Microsoft Sysinternals provides this advanced telemetry. We will install Sysmon with a robust community configuration and forward its logs to Wazuh.

**Steps to Complete:**
- [x] Download Sysmon and a custom Sysmon XML configuration file to the Windows endpoint.
- [x] Install Sysmon using the configuration file via an Administrator Command Prompt.
- [x] Add the Sysmon Event Channel to the Wazuh agent `ossec.conf`.
- [x] Restart the Wazuh agent.
- [x] Verify Sysmon logs are appearing in the Wazuh Dashboard.

**Screenshots Captured:**
- `sysmon_install.png`: Documented the successful installation of the Sysmon service via PowerShell.
- `sysmon_config.png`: Documented the new `<localfile>` block for Sysmon in the agent configuration.

**Errors Encountered & Resolved:**
- **Error:** When filtering for `rule.groups: sysmon` in the Discover tab, the dashboard returned "No Results".
- **Root Cause:** Wazuh ingests Sysmon logs perfectly, but by default, most routine Sysmon events (like opening a normal program) are scored at Level 3 or below. The Wazuh Dashboard (specifically the `wazuh-alerts-*` index) only displays events that are Level 3 or higher. 
- **Resolution:** Moved directly to Phase 5 to create a custom detection rule that forces a high-level alert for specific Sysmon events, proving the telemetry is flowing.

---

## 🚨 Phase 5: Custom Detection Rules

To maximize the value of our integrations (FIM, Defender, and Sysmon), we need to write custom XML detection rules on the Wazuh Server. These rules tell the Wazuh Manager to flag specific behaviors as high-priority security incidents.

**Steps to Complete:**
- [x] Open the Wazuh Dashboard and navigate to Management ➔ Rules (or edit via terminal).
- [x] Select `local_rules.xml` and open the editor to add the custom rule.
- [x] Write a custom rule to detect when a user opens PowerShell (or any process).
- [x] Save the file and restart the Wazuh Manager via the Dashboard prompt or terminal.
- [x] Trigger the rule and capture the alert in the Dashboard.

**Screenshots Captured:**
- `custom_rule.png`: Documented the `local_rules.xml` update inside the Wazuh Dashboard Editor.
- `custom_alert_dashboard.png`: Captured the flood of Sysmon Process Creation events successfully triggering our custom Level 8 rule.

**Errors Encountered & Resolved (General):**
- **Error:** When attempting to open the `ossec.conf` file on Windows, a "File not found" error occurred.
- **Root Cause:** Incorrectly typing `(x86)` in the file name field instead of navigating to the correct directory path (`C:\Program Files (x86)\ossec-agent\`).
- **Resolution:** Used an administrative PowerShell/Command Prompt to directly open the file by running `notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"`.
- **Error:** When running `Restart-Service -Name WazuhSvc`, PowerShell returned `Cannot stop WazuhSvc service`.
- **Root Cause:** The Wazuh background process occasionally hangs in Windows when attempting to stop gracefully, usually while holding onto log file handles.
- **Resolution:** Force-killed the process using `Stop-Process` and then started the service fresh.

---

## 📝 Phase 6: Final Deliverables & Reporting

You have successfully implemented every technical requirement of the internship task! To submit your work, compile the screenshots and documentation gathered into a formal report.

**Report Structure Checklist:**
- [ ] **A. Deployment architecture:** Create a simple diagram (e.g., in draw.io) showing your Wazuh Server, Kali Linux VM, and Windows VM connected via the Bridged Network.
- [ ] **B. Agent enrollment details:** Include screenshots of your active agents from Phase 1 (`linux_agent_active.png`).
- [ ] **C. FIM configuration:** Include the config screenshots and dashboard alerts from Phase 2 (`windows_fim_config.png`, `fim_alert_dashboard.png`).
- [ ] **D. Windows Defender integration:** Include the Defender `<localfile>` config and the EICAR malware alert from Phase 3 (`defender_config.png`, `defender_alert_dashboard.png`).
- [ ] **E. Sysmon integration:** Include the Sysmon installation terminal and the `<localfile>` config from Phase 4 (`sysmon_install.png`, `sysmon_config.png`).
- [ ] **F. Dashboard screenshots and alerts:** Include your final Custom Detection Rule alert from Phase 5 (`custom_rule.png`, `custom_alert_dashboard.png`).
- [ ] **G. Findings and recommendations:** Write a brief conclusion explaining that Wazuh's default logs are noisy but limited, and that integrating Sysmon + Custom Rules drastically improves visibility into endpoint activities.

**GitHub Repository Integration:**
To upload this to GitHub, upload your original `Wazuh Setup.md` (which covers the initial server deployment) along with this newly completed `Wazuh_Advanced_Monitoring.md` file. Together, they represent a complete, end-to-end guide on deploying and configuring an enterprise-grade Wazuh SOC!
