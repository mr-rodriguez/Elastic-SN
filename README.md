# Welcome to Elastic-SN!

## Contributions

This repository is maintained as a controlled reference guide.

- Suggestions are welcome via Issues or Pull Requests
- No direct commits are permitted
- All changes are reviewed and merged solely by the maintainer

## Build Overview

This build walks through deploying a single-node Elastic Stack environment using a structured, phased approach:

1. **Vagrant Environment Setup**  
   Provision the virtualized lab environment and supporting infrastructure.

2. **Elasticsearch Setup**  
   Deploy and configure Elasticsearch nodes for data ingestion and storage.

3. **Kibana & Security Policy Configuration**  
   Configure Kibana, enable security features, and apply baseline policies.

4. **Fleet Server Deployment**  
   Deploy and configure Fleet Server for centralized agent management.

5. **Elastic Agents, Policies, & Integrations**  
   Enroll agents, define agent policies, and enable required integrations.
