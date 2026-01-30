# Welcome to Elastic-SN!

## Contributions

This repository is maintained as a controlled reference guide.

- Suggestions are welcome via Issues or Pull Requests
- No direct commits are permitted
- All changes are reviewed and merged solely by the maintainer

## Build Overview

This build walks through deploying a single-node Elastic Stack environment using a structured, phased approach:

The build is organized into the following chapters under `/docs`:

1. **Vagrant Environment Setup**  
   - `/docs/01-vagrant-environment`  
   - Provision and configure the virtualized lab infrastructure.

2. **Elasticsearch Deployment**  
   - `/docs/02-elasticsearch`  
   - Install, configure, and validate Elasticsearch nodes.

3. **Kibana & Security Policies**  
   - `/docs/03-kibana-and-policies`  
   - Install and configure Kibana.

4. **Fleet Server Setup**  
   - `/docs/04-setup-fleet-server`  
   - Deploy and configure Fleet Server for centralized agent management.

5. **Elastic Agents & Integrations**
   - `/docs/05-agents-agentpolicies-and-integrations`  
   - Enroll agents, define agent policies, and enable integrations.

7. **Index Template and Index Lifecycle Policy**
   - `/docs/06-index-policies`
   - Add an ILM policy to an index template - fix yellow status indicies

