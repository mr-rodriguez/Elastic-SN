# Setup an Index Template and Index Lifecycle Management Policy

Because we created a single-node cluster, your indices' statuses may come up yellow. Here's an example:
- Click the hamburger menu and go to Dev Console
- Run the following GET request
```json
GET _cat/indices?v
```
- Sample GET response
```text
health status index                                                              uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
yellow  open   .internal.alerts-transform.health.alerts-default-000001            8_qyF-njTR2uKVhZbF1XMw   1   1          0            0       249b           249b         249b
```
- notice that `pri` is `1` and `rep` is `1`
- `pri` stands for "primary shards" and `rep` stands for "replica shards"
- you cannot have the replica of a primary shard on the same elasticsearch node, so there is no need for replica shards on a single-node cluster

Let's fix this:
- we will do two things
	- create an ILM policy (since this is a lab, i want to delete data after a certain amount of time)
	- create an Index Template
		- here, we will set the settings to reduce the replica shards to zero whenever a new index is created with the index pattern `logs-*`

1. *ILM Policy*: In Kibana UI:
	1. Stack Management > Index Lifecycle Policies
		1. Create Policy
		2. Enter a name for the policy (e.g. `my-first-ilm-policy`)
		3. Hot phase
			1. Click the trashcan icon
			2. Click Advanced settings
				1. Toggle off `Use recommended defaults` and `Enable rollover`
		4. Scroll down to the Delete phase
			1. Change `Move data into phase when` to whatever. I did 2 hours for this build.
		5. Save policy
2. *Index Template*: In Kibana UI:
	1. Stack Management > Index Management > Index Templates
		1. Create template
		2. Name your template (e.g. `zero_replica_template`)
		3. Index patterns: `logs-*`
		4. Toggle off `Create data stream`
		5. Toggle off `Set index mode`
		6. Priority: `0`
		7. Click Next until you get to `Index settings (optional)`
		8. Enter the settings in the json below
		9. Click Next until the end and click `Create template`
```json
{
  "index.lifecycle.name": "my-first-ilm-policy",
  "number_of_replicas": 0
}
``` 

**Do the following to fix any current yellow status indices**
```json
// check indices statuses
GET _cat/indices?v

// set replicas to zero for all current indices
PUT _settings/
{
  "number_of_replicas": 0
}

// re-check indices statuses
GET _cat/indicies?v
```

All indices should be green, now.

**Congratulations! You're finished =)**
- Challenge: try adding the Elastic Defend integration to the policy
