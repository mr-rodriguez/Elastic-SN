# Generate Fleet Certificates (do this on elasticsearch node)

Use the `elasticsearch-certutil` command to generate the fleet certificates
- don't be confused about `--ip`. Since the fleet server is on the kibana node, you will enter the ip of your kibana instance.
- the following will ask you to keep or rename the `certificate-bundle.zip` file. You may have one already, so I recommend you rename it to something like `fleet.zip`.
```bash
./bin/elasticsearch-certutil cert --pem -ca elastic-stack-ca.p12 \
--name fleet-server \
--ip <ip-of-fleet-server>
```

Unzip the file
```bash
unzip fleet.zip
```

Copy the files to the Kibana certs path
```bash
scp fleet-server/fleet-server.* vagrant@ip-of-kibana-node:/path/to/kibana/certs/
```

# Add a Fleet Server
**Outputs**

In the Kibana UI, click on the hamburger menu and navigate to Fleet
- Click Settings
- Outputs
	- click the pencil icon to edit the default output
		- the default may currently be set to "localhost"
		- we want it to be the IP address of the elasticsearch node
			- Hosts: https://ip-of-elasticsearch-node:9200
			- Advanced YAML
```yaml
ssl.certificate_authorities: ["/path/to/elasticsearch-ca.pem"]
```

**Add Fleet Server**

Next, we'll add a fleet server using the same kibana node (this is fine for our lab environment).
- use a different node for production

In settings:
- Click `Add Fleet Server`
- Advanced
- Click `Create policy`
- Once agent policy is created, scroll down and choose the *Production* deployment mode
	- this is because we are providing our own self signed certificates
- Next, Specify name and URL
	- URL will be typed as FQDN or IP (we will use IP)
	- example: `https:// ip-of-fleet-server:8220` or `https://example.net:8220`
- Click `Add host`
- Click `Generate service token`
- Copy/paste commands that apply to your operating system to a temporary note pad

Elastic Agent Install parameters
- some flags may not populate
- add the ones below that you do not see
```bash
sudo ./elastic-agent install --url=https://ip-of-fleet-server:8220 \
--fleet-server-es=https://ip-of-elasticsearch-node:9200 \
--fleet-server-host=ip-of-fleet-server \
--fleet-server-service-token=<token> \
--fleet-server-policy=fleet-server-policy \
--certificate-authorities=/path/to/elasticsearch-ca.pem \
--fleet-server-es-ca=/path/to/elasticsearch-ca.pem \
--fleet-server-cert=/path/to/fleet-server.crt \
--fleet-server-cert-key=/path/to/fleet-server.key \
--fleet-server-port=8220 \
--install-servers
```

Navigate to `/home/vagrant`, and then run the install

*Cross you fingers that Agent is healthy!*

Check elastic-agent status
```bash
# use this command to check the health of elastic-agent
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


Next, go to `docs/05-agents-agentpolicies-and-integrations.md`
