# Installing Kibana

Download Kibana from archive
- after tar, copy certs that were scp'd from elasticsearch node to an appropriate path
```bash
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-9.2.4-linux-x86_64.tar.gz
```
```bash
curl https://artifacts.elastic.co/downloads/kibana/kibana-9.2.4-linux-x86_64.tar.gz.sha512 | shasum -a 512 -c -
```
```bash
tar -xzf kibana-9.2.4-linux-x86_64.tar.gz
```
```bash
mv elasticsearch-ca.pem kibana-9.2.4/config/
```
```bash
mv kibana-server.* kibana-9.2.4/config/
```
```bash
cd kibana-9.2.4/
```

Generate encryption keys, then add keys to `kibana.yml`
```bash
./bin/kibana-encryption-keys generate
```

Edit `kibana.yml`
- don't copy/paste this
- simply uncomment these sections and type it out. i had some weird "server.hostname" config error
```bash
vim config/kibana.yml
```

```yaml
server.host: ip-of-kibana-node
server.port: 5601
server.hostname: whatever-you-want
elasticsearch.hosts: ["https://ip-of-elasticsearch-node:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "<password>"
server.ssl.enabled: true
server.ssl.certificate: /path/to/kibana-server.crt
server.ssl.key: /path/to/kibana-server.key
elasticsearch.ssl.certificateAuthorities: ["/path/to/elasticsearch-ca.pem"]

xpack.encryptedSavedObjects.encryptionKey: <key>
xpack.reporting.encryptionKey: <key>
xpack.security.encryptionKey: <key>

```

Create a Kibana keystore:
```bash
./bin/kibana-keystore create
```
- add the password for the `kibana_system` user to the Kibana keystore
```bash
./bin/kibana-keystore add elasticsearch.password
```

Start Elasticsearch if not already (from the elasticsearch node)
```bash
./bin/elasticsearch -d -p pid
```

Start Kibana
```bash
./bin/kibana
```

Go to Kibana web browser using HTTPS to validate certs and access

Next, go to `docs/04-fleet-server.md`
