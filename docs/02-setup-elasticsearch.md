# Firewall Rules and SSH Keys
*Before anything else, setup `ufw` on elasticsearch, and kibana nodes*

### Elasticsearch node
```bash
sudo ufw enable ; sudo ufw allow ssh; sudo ufw allow 9200/tcp; sudo ufw allow 9300/tcp; sudo ufw allow 8220/tcp
```
- port info
	- 9200/tcp: client communication over HTTP
	- 9300/tcp: internal communication between nodes within a cluster
	- 8220/tcp: communication between elastic agents and the fleet server
	- ssh: needed to scp files over to kibana and/or other nodes from elasticsearch node (e.g. certificates)

Generate and copy SSH keys
- cannot use `ssh-copy-id`. must copy/paste `id_rsa.pub` manually to `authorized_keys` file
```bash
# passwordless - just keep pressing enter
ssh-keygen
```
```bash
# copy id_rsa.pub to kibana node at /home/vagrant/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub
```

### Kibana node
```bash
sudo ufw enable ; sudo ufw allow ssh ; sudo ufw allow 5601/tcp ; sudo ufw allow 8220/tcp
```
- port info
	- 5601/tcp: browser > kibana ui
	- 8220/tcp: fleet > elasticsearch
	- ssh: same as with elasticsearch node

Generate and copy SSH keys
- cannot use `ssh-copy-id`. must copy/paste `id_rsa.pub` manually to `authorized_keys` file
```bash
# passwordless - just keep pressing enter
ssh-keygen
```
```bash
# copy id_rsa.pub to elasticsearch node at /home/vagrant/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub
```
# Installing Elasticsearch

## Download Elasticsearch from archive
```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.2.4-linux-x86_64.tar.gz
```
```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.2.4-linux-x86_64.tar.gz.sha512
```
```bash
shasum -a 512 -c elasticsearch-9.2.4-linux-x86_64.tar.gz.sha512
```
```bash
tar -xzf elasticsearch-9.2.4-linux-x86_64.tar.gz
```
```bash
cd elasticsearch-9.2.4/
```

## Set up node for connectivity
Configure `elasticsearch.yml`
```bash
vim config/elasticsearch.yml
```

```YAML
# cluster name not needed unless you add more elasticsearch nodes to your cluster
# cluster.name: whatever-you-want
network.host: ip-of-elasticsearch-node
http.port: 9200
node.name: whatever-you-want
discovery.type: single-node
```

Start Elasticsearch
- I decided to run elasticsearch as a daemon
```bash
bin/elasticsearch -d -p pid

# command to kill the process (must be in folder where pid file is located)
# pkill -F pid
```

Set passwords ahead of time
- save them somewhere temporarily to use for validation purposes
- the kibana_system username will be used when configuring the kibana instance
```bash
bin/elasticsearch-reset-password -i -u elastic
```
```bash
bin/elasticsearch-reset-password -i -u kibana_system
```

Check that elasticsearch is up and running ok (from a different node on the network)
```bash
curl -XGET -k -u elastic https://ip-of-elasticsearch-node:9200/_cluster/health?pretty
```

Stop and reconfigure elasticsearch
- when elasticsearch first runs, security configs are added to `elasticsearch.yml`
- we'll need to make some adjustments
```bash
pkill -F pid
```

## Minimal security setup
Edit `elasticsearch.yml`
```bash
vim config/elasticsearch.yml
```

Comment out `cluster.initial_master_nodes: ["<node_name>]` 
- if both are uncommented, you will get an error when starting elasticsearch
```YAML
discovery.type: single-node
#cluster.initial_master_nodes: ["<node_name>"]
```
## Setup transport TLS: Generate certs (do this on elasticsearch node)
```bash
./bin/elasticsearch-certutil ca
```
```bash
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```
- this creates `elastic-certificates.p12`

Move it
```bash
mv elastic-certificates.p12 config/
```

If you entered a password when creating the node certificate, run these commands to store the password in the Elasticsearch keystore
- when prompted to overwrite, say Y for yes and enter the same password you used when creating the node certs
```bash
./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
```

```bash
./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```


Edit `elasticsearch.yml` to how you see below
- leave `xpack.security.http.ssl.keystore.path:` as is it for now
```bash
vim config/elasticsearch.yml
```
```YAML
xpack.security.enabled: true
xpack.security.http.ssl:
	enabled: true
	keystore.path: <wait until http cert is created>
	
xpack.security.transport.ssl:
	enabled: true
	verification_mode: certificate
	client_authentication: required
	keystore.path: elastic-certificates.p12
	truststore.path: elastic-certificates.p12
```

## Setup HTTPS

Generate TLS certs with Elasticsearch HTTP cert tool
```bash
./bin/elasticsearch-certutil http
```
- This command generates a `.zip` file that contains certificates and keys to use with Elasticsearch and Kibana. Each folder contains a `README.txt` explaining how to use these files.
    
    1. When asked if you want to generate a CSR, enter `n`.
        
    2. When asked if you want to use an existing CA, enter `y`.
        
    3. Enter the path to your CA. This is the absolute path to the `elastic-stack-ca.p12` file that you generated for your cluster.
        
    4. Enter the password for your CA.
        
    5. Enter an expiration value for your certificate. You can enter the validity period in years, months, or days. For example, enter `90D` for 90 days.
        
    6. When asked if you want to generate one certificate per node, enter `n`.
        
        Each certificate will have its own private key, and will be issued for a specific hostname or IP address.
        
    7. When prompted, enter the name of the first node in your cluster.
        
    8. Enter all hostnames used to connect to your first node. These hostnames will be added as DNS names in the Subject Alternative Name (SAN) field in your certificate.
        
        List every hostname and variant used to connect to your cluster over HTTPS.
        
    9. Enter the IP addresses that clients can use to connect to your node.
        
    10. Repeat these steps for each additional node in your cluster.
        
- After generating a certificate for each of your nodes, enter a password for your private key when prompted.
- Unzip the generated `elasticsearch-ssl-http.zip` file. This compressed file contains one directory for both Elasticsearch and Kibana.
```txt
/elasticsearch
|_ README.txt
|_ http.p12
|_ sample-elasticsearch.yml
```

*IGNORE THIS DIRECTORY AND PEM FILE. WE WILL CREATE ONE USING THE `http.p12` above*
```txt
/kibana
|_ README.txt
|_ elasticsearch-ca.pem
|_ sample-kibana.yml
```

- Move the cert to the right location
```bash
mv elasticsearch/http.p12 config/
```

- Now edit the `elasticsearch.yml` file to enable HTTPS security and specify the location of the `http.p12` security certificate
```yaml
xpack.security.http.ssl.keystore.path: http.p12
```

- Add the password for your private key to the secure settings in Elasticsearch
```shell
./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
```

Extract CA from Elasticsearch `http.p12`
```bash
openssl pkcs12 -in config/http.p12 -cacerts -nokeys -out elasticsearch-ca.pem
```

Copy `elasticsearch-ca.pem` to Kibana (which will also be our Fleet server)
- *At this point, we have not installed kibana yet, so copy this over to home directory*
- DO NOT USE the auto-generated `elasticsearch-ca.pem` that's in the `kibana/` folder mentioned above
```bash
scp elasticsearch-ca.pem vagrant@ip-of-kibana:/home/vagrant
```

Start Elasticsearch

## Encrypt traffic between your browser and kibana
```bash
./bin/elasticsearch-certutil cert --pem -ca elastic-stack-ca.p12 -name kibana-server
```
- This command generates a `certificate-bundle.zip` file by default with the following contents:
```txt
/kibana-server
|_ kibana-server.crt
|_ kibana-server.key
```

Unzip `certificates-bundle.zip` to obtain the certs, and then copy over to the kibana node where you keep your certs
- *again, at this point we have not installed kibana yet. copy to home dir*
```bash
unzip certificate-bundle.zip
```
```bash
scp kibana-server/kibana-server.* vagrant@ip-of-kibana:/home/vagrant
```

Done for now

*At this point, I created a snapshot of the vagrant box*
- must be done on local machine, not in the vagrant box
```bash
vagrant snapshot save --timestamp elasticsearch01 elasticsearch01-post-step1-config
```

- Go to `docs/03-kibana-and-policies.md` 
