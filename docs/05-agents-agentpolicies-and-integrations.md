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
- create a directory path that matches what you entered in Fleet > Settings > Outputs > `Advanced YAML`
  - in our case it was `/home/vagrant/kibana-9.2.4/config/`
  - this feels weird, but i ran into many issues here, so this is to keep it simple
- copy the `elasticsearch-ca.pem` to that directory
- you will need to create an ssh key and copy it to the elasticsearch node's `authorized_keys`
```bash
scp vagrant@10.10.1.2:/home/vagrant/elasticsearch-9.2.4/elasticsearch-ca.pem /home/vagrant/kibana-9.2.4/config/
```


# Create and configure policy and add elastic agent

**Step 1.**
- In Kibana UI:
  - Hamburger menu > Fleet > Agent Policies
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
  	- add the install flags shown below to your list of install commands
  	- then, run the install
```text
--certificate-authorities=/path/to/elasticsearch-ca.pem \
--fleet-server-es-ca=/path/to/elasticsearch-ca.pem 
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

Check here, too
```bash
sudo systemctl status elastic-agent
```

Check Fleet > Agents
- The agent now shows `Healthy` for me

**Step 5.**
- In Kibana UI
	- Check Auditd Logs dashboard to see if any data is populating
		- Analytics > Dashboards > `[Logs Auditd] Audit Events`
    - you should see data. if not, try running some privileged commands on the box to generate logs, then refresh the dashboard page

Next, go to `docs/06-index-policies`
