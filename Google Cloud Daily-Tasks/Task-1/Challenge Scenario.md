# Challenge scenario.
 - ### As a cloud engineer at Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

   #### *You need to complete the following tasks:*
    - Create a development VPC with three subnets manually.
    - Create a production VPC with three subnets manually.
    - Create a bastion that is connected to both VPCs.
    - Create a development Cloud SQL Instance and connect and prepare the WordPress environment.
    - Create a Kubernetes cluster in the development VPC for WordPress.
    - Prepare the Kubernetes cluster for the WordPress environment.
    - Create a WordPress deployment using the supplied configuration.
    - Enable monitoring of the cluster.
    - Provide access for an additional engineer.

# Architecture

```text
                    Internet
                        │
                        ▼
                Load Balancer Service
                        │
                        ▼
               GKE Cluster (griffin-dev)
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
 WordPress Pod                 Cloud SQL Proxy
        │                               │
        └───────────────┬───────────────┘
                        ▼
              Cloud SQL MySQL Instance
                    griffin-dev-db

Networks:
---------
griffin-dev-vpc
 ├── griffin-dev-wp
 └── griffin-dev-mgmt

griffin-prod-vpc
 ├── griffin-prod-wp
 └── griffin-prod-mgmt

Bastion Host
 ├── Dev Mgmt NIC
 └── Prod Mgmt NIC
```



















