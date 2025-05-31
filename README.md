# SOAR EDR Project

## Table of Contents

- [Design](#Desgin)
- [Install LimaCharlie](#install-limacharlie)
- [Create Threat Simulation with Lazagne](#create-threat-simulation-with-lazagne)


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


## Install LimaCharlie

### Step 1: Install Windows Server

First, we need a Windows Server machine to act as an agent. In this guide, we'll use VMware.

- Download the Windows Server ISO image from [this link](https://info.microsoft.com/ww-landing-windows-server-2019.html).
- In VMware, go to `File -> New Virtual Machine`.
- Choose your hardware settings and select the ISO image you downloaded.
- Start the virtual machine, and follow the installation wizard by clicking "Next" through the prompts.

> The Windows Server installation process is straightforward, and final will end up with windows server like this.

![windows-server](windwos-server.png)

---

### Step 2: Sign Up on LimaCharlie

Now, create an account on [LimaCharlie](https://limacharlie.com).

- After signing up and creating an organization, you’ll see a dashboard like the one below:  
  ![LimaCharlie Dashboard](limadashboard.png)

- Navigate to `Sensors -> Installation Keys` and click **Create Installation Key**.

> You can delete any other keys you don’t need.

![Installation Key Page](installation.png)

---

### Step 3: Install the LimaCharlie Agent

- On the same `Installation Keys` page, scroll down to **Sensor Downloads**.
- Choose `Windows 64-bit` under the EDR section and copy the download link.
- On your Windows Server, paste the link into a browser to download the agent.

![Sensor Download Section](Sensors_down.png)

- Open the PowerShell window in the folder where you downloaded the agent.
- Run the installation command:
  ![PowerShell Installation Command](powercommand.png)

> Use the **Sensor Key** you created earlier under your Installation Key settings.

---

### Step 4: Verify the Agent

#### Option 1: Windows Services
Check that the **LimaCharlie service** is running via the Windows Services manager:

![Windows Services Showing LimaCharlie](limaservice.png)

#### Option 2: LimaCharlie Dashboard
Alternatively, go to `Sensors -> Sensors List` in the LimaCharlie dashboard. You should see your Windows Server listed as a connected sensor.

![Sensor List on Dashboard](sensorlist.png)


## Create Threat Simulation with Lazagne

### Step 1: Download the Hack Tool

We will use **Lazagne**, a credential dumping tool, as our simulated attack.

- Download Lazagne on the Windows Server machine.
- You can find it [here](https://github.com/AlessandroZ/LaZagne) (use the precompiled binary).
- **Important:** Disable Windows Defender before running it, as it will block the tool.

Once downloaded, open **PowerShell** and run Lazagne like this:

![Lazagne Execution](lazagne.png)

---

### Step 2: Detect the Tool in LimaCharlie

Now, switch to the **LimaCharlie dashboard** to observe if the tool activity was detected:

1. Go to `Sensors -> Sensor List`.
2. Click on your Windows Server machine.
3. In the sidebar, go to the **Timeline** tab.
4. Search for `lazagne`.

You should see events like this:

![Lazagne Timeline Events](lazagne-event.png)

---

### Step 3: Create a Detection Rule

To build a detection rule specific to Lazagne, go to:

```
Automation -> D&R Rules
```

After reviewing the documentation and with help from ChatGPT, I created a rule that has two parts: **Detection Logic** and **Response Metadata**.

#### 🧠 Detection Logic

```yaml
events:
  - NEW_PROCESS
  - EXISTING_PROCESS
op: and
rules:
  - op: is windows
  - op: or
    rules:
      - case sensitive: false
        op: ends with
        path: event/FILE_PATH
        value: lazagne.exe
      - case sensitive: false
        op: contains
        path: event/COMMAND_LINE
        value: lazagne.exe
      - case sensitive: false
        op: is
        path: event/HASH
        value: 'dc06d62ee95062e714f2566c95b8edaabfd387023b1bf98a09078b84007d5268'
```

#### ⚙️ Response Metadata

```yaml
- action: report
  metadata:
    - author: msferhet
    - description: Detect Lazagne (SOAR EDR Tool)
    - falsepositive: to moon
    - level: medium
  tags:
    - attack credential access
  name: detect - hacktool - lazagne (SOAR EDR Tool)
```

---

### Step 4: Test the Rule

Once the rule is created, scroll down and click **Create**.

Then:

1. Under the rule, find and click on **Target Event**.
2. Go back to the **Timeline**, find a Lazagne-related event.
3. Copy the event JSON and paste it into the test field.
4. Click **Test Event**.

![Test Event](Test-event.png)

---

### Step 5: Validate Detection

Re-run Lazagne on your Windows Server.

Then:

1. Go back to LimaCharlie.
2. Navigate to the **Detection** tab.
3. Wait a moment — you should see a detection alert like this:

![Detection Alert](detection.png)

✅ **Success!** Your custom rule is now actively detecting malicious tool usage.