# SOAR EDR Project

## Table of Contents

- [Design](#Desgin)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Contributing](#contributing)
- [License](#license)


## Desgin

![Soar EDR Playbook](playbook.png)

###  Workflow Sequence

1. **Threat Detection**
   - A hacking tool or suspicious activity is detected on an endpoint by **LimaCharlie (EDR)**.

2. **Generate Alert**
   - LimaCharlie sends an **alert** to the **Tines (SOAR)** platform.

3. **Orchestrate Response**
   - Tines begins an automated response workflow.
   - It sends a **detailed message** to:
     - **Slack** (for analyst visibility)
     - **Gmail** (for email-based alerting)

   **Message Details Include:**
   - Time  
   - Computer Name  
   - Source IP  
   - Process  
   - Command Line  
   - File Path  
   - Sensor ID  

4. **Analyst Decision: Isolate Machine?**
   - Tines prompts the analyst (e.g., via Slack) with a question:
     > _“Do you want to isolate the affected machine?”_

5. **If the Analyst Responds "Yes":**
   - Tines sends a command to **LimaCharlie** to **isolate the compromised machine**.
   - LimaCharlie performs the isolation action.
   - A confirmation message is sent to **Slack**:
     - _"Computer `<computer>` isolated."_

6. **If the Analyst Responds "No":**
   - Tines sends a message to **Slack** instructing:
     - _"Please investigate."_
   - No isolation is performed; the alert awaits manual follow-up.
