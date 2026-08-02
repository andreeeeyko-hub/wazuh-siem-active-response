# wazuh-siem-active-response

# Wazuh SIEM Lab: Custom Detection Rules & Active Response Playground

## About The Project
This is a hands-on learning lab created to experiment with **Wazuh SIEM**, custom rule creation, security event visualization, and automated threat response. 

The main goal was to set up a testing environment, generate real-world logs, and see how a SIEM detects and reacts to system actions in real time.

---

##  Lab Setup & Stack
* **SIEM Core:** Wazuh Manager & Wazuh Dashboard
* **Endpoints:** Linux (Kali Linux / Ubuntu) and Windows Workstations
* **Key Features Tested:**
  * Endpoint log ingestion (`/var/log/auth.log` & Windows Event Viewer)
  * Custom XML detection rules mapped to **MITRE ATT&CK**
  * Automated IP blocking via **Active Response**
  * Custom dashboard visual widgets & data tables

---

##  Step 1: Writing a Custom Rule

To test custom alerting, I created a rule to track `sudo` command execution (privilege escalation activity) on the Linux agent.

File location: `/var/ossec/etc/rules/local_rules.xml`

```xml
<group name="local,syslog,sudo,">
  <!-- Custom Alert for tracking SUDO execution -->
  <rule id="100001" level="7">
    <if_sid>5402</if_sid>
    <description>Custom Alert: User executed a command with SUDO privileges</description>
    <mitre>
      <id>T1078</id>
    </mitre>
    <group>privilege_escalation,</group>
  </rule>
</group>
```
Rule ID 100001: Assigned in the custom rule ID range (100000–120000).

Level 7: Elevated priority to make alerts stand out on the dashboard.

MITRE ATT&CK: Mapped to technique T1078 (Valid Accounts).

## Step 2: Configuring Active Response (Auto-Block)
To test automated response actions, I configured the Active Response module to run the built-in firewall-drop script when the rule triggers.

File location: /var/ossec/etc/ossec.conf

```xml
<ossec_config>
  <command>
    <name>firewall-drop</name>
    <executable>firewall-drop</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rule_id>100001</rule_id>
    <timeout>60</timeout>
  </active-response>
</ossec_config>
```
When Rule 100001 triggers, the agent automatically drops traffic using iptables for 60 seconds.

## Step 3: Simulation & Dashboard Setup
Testing: Executed several sudo commands on the Linux agent terminal (sudo -k && sudo whoami).

SIEM Results:

Events were captured and filtered using rule.id: 100001.

Created a custom SUDO Alerts table displaying agent.name and raw command details (full_log).

![Wazuh Dashboard](docs/screenshots/dashboard)

