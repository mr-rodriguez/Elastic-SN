# Generate Fleet Certificates (do this on elasticsearch node)

Use the `elasticsearch-certutil` command to generate the fleet certificates
- don't be confused about `--dns`. Since the fleet server is on the kibana node, you will enter the hostname of your kibana instance. Same this with `--ip`.
- the following will ask you to keep or rename the `certificate-bundle.zip` file. You may have one already, so I recommend you rename it to something like `fleet.zip`.
```bash
./bin/elasticsearch-certutil cert --pem -ca elastic-stack-ca.p12 \
--name fleet-server \
--dns <hostname-of-fleet-server> \
--ip <ip-of-fleet-server> \
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
--fleet-server-service-token=<token> \
--fleet-server-policy=fleet-server-policy \
--certificate-authorities=/path/to/elasticsearch-ca.pem \
--fleet-server-es-ca=/path/to/elasticsearch-ca.pem \
--fleet-server-cert=/path/to/fleet-server.crt \
--fleet-server-cert-key=/path/to/fleet-server.key \
--fleet-server-port=8220 \
--install-servers
```

*Cross you fingers that Agent is healthy!*

Check elastic-agent status
```bash
# use this command to check the health of elastic-agent
sudo /opt/Elastic/Agent/elastic-agent status
```

*I got errors!*
- elastic-agent -> healthy
- fleet -> communication error

Let's troubleshoot!
```bash
ss -tulnp
```

- I see a local address with the port number 8221
- the errors I saw earlier eluded to localhost:8221. this confused me
- I checked the Outputs in kibana ui fleet settings, and the elasticsearch node was set correctly and not to localhost:8221
- I checked the `/home/vagrant/kibana-9.2.4/elastic-agent.yml` and *found the culprit*

```yaml
######################################
# Fleet configuration
######################################
outputs:
  default:
    type: elasticsearch
    hosts: [127.0.0.1:9200]
    api_key: "example-key"
    #username: "elastic"
    #password: "password"
    preset: balanced
```

Let's fix this!

Stop elastic-agent service
```bash
sudo systemctl stop elastic-agent
```

Create API key in Kibana UI
- Stack Management > API keys
- Create API key (top right button)
	- Name your key. I put `fleet-key`
	- Click `Create API key`
	- **Important**: copy the encoded key and save it! you will not be able to view it again

Edit `/home/vagrant/kibana-9.2.4/elastic-agent.yml`
```bash
######################################
# Fleet configuration
######################################
outputs:
  default:
    type: elasticsearch
    hosts: [ip-of-elasticsearch-node:9200]
    api_key: "the-api-key-you-just-copied-goes-here"
    username: "elastic"
    password: <password>
    preset: balance
```

Start the elastic agent service
```bash
sudo systemctl start elastic-agent
```

Check elastic agent status
```bash
sudo /opt/Elastic/Agent/elastic-agent status
```
```text
┌─ fleet
│  └─ status: (HEALTHY) Connected
└─ elastic-agent
   └─ status: (HEALTHY) Running
```

Next, go to `docs/05-agents-agentpolicies-and-integrations.md`
