# Google Professional Cloud Architect

## Decisions

When trying to understand scenario based questions it helps to break the question down to keyword that define the solution.

- Data < 10TB & single region — Cloud SQL
- Data > 10TB — BigQuery
- Transactional NoSQL database — Cloud Datastore
- User Profile and Game State — Cloud Datastore
- Timeseries database — Bigtable
- Streaming/arrives late/Buffers data — Pub/Sub
- Analyze and optimize for performance — Stackdriver
- Automation framework — Deployment manager/Terraform
- Hadoop/Spark — Dataproc
- Convert onprem HDFS/Object Storage — Cloud Storage first HDFS second
- Block Storage — Persistent Disk
- Containers — Kubernetes
- Network protocols other than HTTPS — Kubernetes or Compute Engine
- Websites — App Engine
- Static websites — Cloud Storage
- Event Driven — Cloud Functions
- Relational Data — Cloud SQL (vertical scaling) or Spanner (horizontal scaling)
- Analytics Database — BigQuery
- In memory — Cloud Memorystore (Redis)
- No-Ops/Managed service — No infrastructure to manage such as GKE and GAE.
- Data Lake/HDFS — Cloud Storage
- Preemptible VMs — Lives only for 24 hours

StackDriver

- Monitoring: Dashboard, Workspace, Alert
- Logging
- Error Reporting
- Debugger
- Trace

Compute:

- Compute Engine
- GKE
- App Engine
- Cloud Function: from http, pub/sub, storage
- Cloud Run
- Cloud Schedule: to HTTP, App engine, Pub/Sub
- Cloud Task: Queue to App Engine for long-term job
- Shielded VM

Network

- Cloud VPC peering: VPC to VPC
- Direct Peering: on-prem to google point of presence, public ip
- Cloud VPN: multiple private connection (3gbps)
- Compute Engine only have private ip, but wanna access public
  - Google Private Access for Subnet
  - Cloud NAT Gateway
- Cloud Interconnect: bandwidth and latency requirement, on-prem to google
  - Dedicted interconnect: > 10Gbps
  - Partner Interconnect: < 10 Gbps
  - private ip
  - have to change original env
- Gate egress 私有存公有, Gate ingress 私有存公有
- VPC Service Controls allow customers to address threats such as data theft, accidental data loss, and excessive access to data stored in Google Cloud multi-tenant services. It enables clients to tightly control what entities can access what services in

Core & Security:

- key management: CSEK(Cloud stroage and compute), CMEK using cloud KMS
- compliance: HIPAA, PCI-DS, GDPR, COPPA(US Child GDPR), SOX (financial)
- TCO calculations, OpEx/CapEx allocation

Tools

- Cloud composer: Apache Airflow
- Google Cloud Directory Sync

Database & Storage

- BigQuery: expiration, partitiontimestamp
- dataproc: hardoop, spark, java, regional
- dataflow: xxxx to bigquery
- Data Studio: visualize data from bigquery
- Data Lab: jupyter

## Tips

- There are totally 50 questions and you have 120 minutes — so you have around 2.5 minutes for each question. So take your time to read the question carefully before choosing the answer and also review each question once you completed all questions.
- Read questions properly, sometimes they are tricky and you can often miss some minute details which will be a key point to find the correct answer.
- Case studies covers around 20–25% of the questions (~10–14 questions). So, please concentrate more on the 3 case studies given and remember the solutions for them.
- If you take the exam from home, then you are video proctored. So strictly follow the Webassessor guide for taking the exam.

## Topics

Section 1: Designing and planning a cloud solutions architecture

1.1 Designing a solutions infrastructure that meets **business requirements**
1.2 Designing a solutions infrastructure that meets **technical requirements**
1.3 Designing **network**, **storage**, and **compute** resources
1.4 Creating a **migration plan** (i.e., documents and architectural diagrams)
1.5 Envisioning future **solutions improvements**

Section 2: Managing and provisioning solutions Infrastructure

2.1 Configuring network topologies
2.2 Configuring individual storage systems
2.3 Configuring compute systems

Section 3: Designing for security and compliance

3.1 Designing for security
3.2 Designing for legal compliance

Section 4: Analyzing and optimizing technical and business processes

4.1 Analyzing and defining technical processes
4.2 Analyzing and defining business processes
4.3 Developing procedures to test resilience of solutions in production (e.g., DiRT and Simian Army)

Section 5: Managing implementation

5.1 Advising development/operation team(s) to ensure successful deployment of the solutions
5.2 Interacting with Google Cloud using GCP SDK (gcloud, gsutil, and bq)

Section 6: Ensuring solutions and operations reliability

6.1 Monitoring/logging/alerting solutions
6.2 Deployment and release management
6.3 Supporting operational troubleshooting
6.4 Evaluating quality control measures

## Cases

### Mountkirk Games

- online, session-based, multiplayer games for mobile platforms
- scaling their global audience, application servers, MySQL databases, and analytics tools.
- write game statistics to files and send them through an ETL tool that loads them into a centralized MySQL database for reporting.
  - ETL: Extract-Transform-Load
- They plan to deploy
  - the game’s backend on Compute Engine so they can capture streaming metrics, run intensive analytics
  - take advantage of its autoscaling server environment
  - integrate with a managed NoSQL database
- Business requirements  
  - Increase to a global footprint
  - Improve uptime—downtime is loss of players
  - Increase efficiency of the cloud resources we use
  - Reduce latency to all customers
- Requirements for game backend platform
  - Dynamically scale up or down based on game activity
  - Connect to a transactional database service to manage user profiles and game state
  - Store game activity in a timeseries database service for future analysis
  - As the system scales, ensure that data is not lost due to processing backlogs
  - Run hardened Linux distro
- Requirements for game analytics platform
  - Dynamically scale up or down based on game activity
  - Process incoming data on the fly directly from the game servers
  - Process data that arrives late because of slow mobile networks
  - Allow queries to access at least 10 TB of historical data
  - Process files that are regularly uploaded by users’ mobile devices

### TerramEarth

- solution concept
  - 20 million TerramEarth vehicles in operation that collect 120 fields of data per second
  - Data is stored locally on the vehicle and can be accessed for analysis when a vehicle is serviced
  - 200,000 vehicles are connected to a cellular network, allowing TerramEarth to collect data directly.
  - At a rate of 120 fields of data per second, with 22 hours of operation per day
  - TerramEarth collects a total of about 9 TB/day from these connected vehicles.
- Existing technical environment
  - Linux and Windows-based systems
  - gzip CSV files from the field and upload via FTP
  - aggregated reports are based on data that is three weeks old
  - With this data, reduce unplanned downtime of their vehicles by 60%
  - because the data is stale, some customers are without their vehicles for up to four weeks while they wait for replacement parts
- Business requirements
  - Decrease unplanned vehicle downtime to less than one week
  - Support the dealer network with more data on how their customers use their equipment to better position new products and services
  - Have the ability to partner with different companies
- Technical requirements
  - Expand beyond a single data center to decrease latency to the American Midwest and East Coast
  - Create a backup strategy
  - Increase security of data transfer from equipment to the data center 
- Applications
  - Data ingest
  - Reporting
- Executive statement

### Dress4Win

- background
  - web-based company that helps their users organize and manage their personal wardrobe using a web app and mobile application
- solution concept
  - migration to the cloud. development and test environments
  - disaster recovery site
- Existing technical environment
  - DB: MySQL
  - 40 web java nginx
  - 20 hadoop
  - 3 rabbitmq
  - CI
  - mysql storage 1PB
  - iSCSI VM
  - image/log and backup 100TB
- Business requirements
  - reliable and reproducible environment with scaled
  - Secure: Improve security by defining and adhering to a set of security and identity and access management (IAM) best practices for cloud
  - business agility and speed of innovation
  - Analyze and optimize architecture for performance in the cloud
- Technical requirements
  - Easily create non-production environments in the cloud
  - automation framekwork
  - Implement a continuous deployment process for deploying applications to the on-premises data center or cloud
  - failover of the production environment 
  - Encrypt data on the wire and at rest 
  - multiple private connections between the production data center and cloud environment.
- Executive Statement
  - Our traffic patterns are highest in the mornings and weekend evenings; during other times, 80% of our capacity is sitting idle.
  - total cost of ownership (TCO)

## Reference

- https://medium.com/@dynamicbalaji/google-cloud-platform-professional-cloud-architect-gcp-pca-certification-preparation-guide-4b8a85ceea93
- https://google.qwiklabs.com/
- https://raw.githubusercontent.com/gregsramblings/google-cloud-4-words/master/DarkPoster-medres.png
- labs: https://www.qwiklabs.com
- quests: https://www.whizlabs.com/learn/course/gcc-pca-pt/
