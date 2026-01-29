# Important!

Since we are using self signed certificates, you must do either of the following
- Option A: set up a proxy server that holds the elasticsearch certificates
- Option B: add the elasticsearch ca certificate to each endpoint you plan to add to fleet
  - I am doing option b since this is a demo for 1 agent

# Install Auditd on test linux workstation

Let's add an ubuntu device to our fleet server
- We will add the auditd integration to the agent policy

**Step 1.**
- ensure auditd is on the linux box
```bash
sudo apt update && sudo apt install auditd audispd-plugins
```
```bash
sudo systemctl enable --now auditd && sudo systemctl status auditd
```

**Step 2.**
- create a directory that matches the path you configured in the Fleet Outputs under `Advanced YAML`
	- then, copy the `elasticsearch-ca.pem` from your elasticsearch node over to that path

# Create and configure policy and add elastic agent

**Step 1.**
- In Kibana UI:
  - Stack Management > Fleet > Agent Policies
  - Create policy (let's name it `ubuntu-agent-policy`)
  	- leave defaults as is
  - Click on your newly created policy
  - Click the `system-` integration policy
  	- Toggle off `Collect events from the Windows event log`
  	- leave everything else as is
  	- Save integration
  - Click `+ Add integration`
  	- select `Auditd Logs` integration
  	- click `Add integration`
  - Go back to Fleet > Agents and click `Add agent`
  	- Choose the agent policy you created from the drop down
  	- Enroll in Fleet
  	- Choose your OS arch and copy/paste commands to a text editor
  	- add the following install flags to your list of install commands
  	- then, run the install
```text
--certificate-authorities=/path/to/elasticsearch-ca.pem \
--fleet-server-es-ca=/path/to/elasticsearch-ca.pem \
```

**Step 2.**
- check the health status
```bash
sudo /opt/Elastic/Agent/elastic-agent status
```
```text
┌─ fleet
│  └─ status: (HEALTHY) Connected
└─ elastic-agent
   └─ status: (HEALTHY) Running
```

**Step 5.**
- In Kibana UI
	- Check Auditd Logs dashboard to see if any data is populating
		- Analytics > Dashboards > `[Logs Auditd] Audit Events`
    - may need to run some privileged commands on the box to generate logs

**Done! Congratulations =)**
- now try adding the Elastic Defend integration to your ubuntu policy!
