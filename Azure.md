# Azure

## Misc notes to organize later

- Dashboard is a custom way of organizing everything. You can also pin resources there, and move things around by clicking the Edit button.
- Sometimes creating a resource will create multiple resources at once, e.g. creating an "app service" resource automatically creates an "app service plan" resource.

- ARM templates: Azure Resource Management.
  - Infrastructure as Code.
  - only used for deploying infrastructure.
  - JSON format.
  - can be comprised of single or multiple files.
  - requires resources to be deployed in the proper order.
  - can be deployed through: Azure portal, CI/CD pipelines, Rest API, and more.
- Bicep:
  - alternative to ARM templates, since ARM templates can be very complicated.
  - JSON format.
  - advantages over ARM:
    - simpler to write.
    - don't have to worry about deployment order (ARM does require proper deployment order).
    - modular: can mix and match between other Bicep files.
    - allow you to preview changes before making them.

- resource names have to be globally unique. Microsoft has many naming conventions that can be found [here]([LO4cGdmd1U^ns4PS3qEq](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)).
- resource locations typically should be somewhere near you. there are 3 "West US" options:
  - West US: California
  - West US 2: Washington
  - West US 3: Arizona

## Hierarchy in Azure

1. management groups
   - manages policy.
2. subscriptions
   - associate user accounts with the resources that they own. So, if you as a user have access to create resources in a subscription, that subscription will show up as an option in the UI when creating a new resource.
3. resource groups
   - a place where like-minded services live.
4. resources

## Amounts of Responsibility

- on prem:
  - you scale and manage everything:
    - applications
    - data
    - runtime
    - middleware
    - OS
    - virtualization
    - servers
    - storage
    - networking
- Infrastructure as a service (IaaS):
  - you scale and manage the following:
    - applications
    - data
    - runtime
    - middleware
    - OS
  - advantages: reduces hardware cost, better optimization, stability, SLA, enhanced security.
- Platform as a Service (PaaS):
  - you scale and manage the following:
    - applications
    - data
  - advantages: reduced development time, more capability with less staff, multi-platform tooling (desktop, web, mobile), ability to experiment with tooling and not worry about cost (pay as you go), lifecycle management (seemless CI/CD since so much of it is plug and play).
- Serverless:
  - you split responsibilities for the following (no scaling):
    - applications
    - data
  - advantages: zero infrastructure management, zero scaling management, reduced time to market, low cost of ownership
- Software as a Service (SaaS):
  - Azure takes care of everything.
  - advantages: applications already built, 

## Webpage Services

Note: Project will be housed in Azure DevOps.

### App Service Plan (ASP)

- service dependencies:
  - requires: none
  - required by: App Service
- can host many App Services.
- configures resources used to build the web app such as memory and CPU speed.
- determines OS, region, zone redundancy, and pricing tier.

### App Service

- service dependencies:
  - requires: App Service Plan (ASP)
  - required by: App Service
- holds Azure Web Apps
- PaaS
- advantages:
  - support for multiple languages and frameworks
  - easily integrate with DevOps
  - supports multiple environments
  - built-in SaaS connectors, e.g. connect to Saleforce with a premade container
  - supports containerization e.g. you can use a Docker image as part of your hosting process
  - support for many APIs and mobile features e.g. push notifications
  - built in security and compliance

### Web App

- service dependencies:
  - requires: App Service
  - required by: App Service

### Azure Blob Storage

Hosting method for static files.

### Content Delivery Network (CDN)

Serves files from Blob Storage.

### Azure Database

todo

### Some other Database thing: Logical Server

todo

### API Management

Helps secure and manage API endpoints.

### API App

Creates an API. .NET supported and extremely easy to integrate. once created:
- enter the service
- click deployment center on the left menu
- here under "Source" are different ways you can deploy your code. Select "Local Git", then "Save".
- Copy the "Git Clone Uri".
- in your local API project, move into the root directory
- `git remote add azure [TheGitCloneUri]`
- `git push azure master`
- In Azure, next to the "Settings" and "Logs" tab there should be a "Local Git/FTPS credentials" tab. Select that and use the Application scope "Local Git Username" and "Password" fields as your git credentials.
- Click "Overview" from the left menu, then the link under "default domain". This is the page where your API will appear. Remember to add `/swagger/index.html` to the end of the url if needed.

### Azure Cache For Redis

Caches some database requests and improves speed.

### App Gateway

Helps with web traffic and can put up firewall to keep out malicious attackers

### Azure Key Vault

Stores secrets.

### Azure App Config

Stores "configurations".

### Log Analytics

Keeps track of logs.

### Azure Monitor

Keeps track of logs.

### Application Insights

Picks up relevant logs from services and "pivot" on that data.

## Understanding VMs

### Terminology:

- virtual CPU (vCPU): basically the same as a CPU.
- Azure Compute Unit (ACU): the amount of computing power each VM has.
- Azure Disk Storage: virtual storage for the VMs. depending on the type, can hold up to 64TB.
- Azure Virtual Machine Scale Sets: handle load balancing and autoscaling for VMs. Typically multiple VMs will do hosting for production. Virtual Machine Scale set let you configure and deploy new VMs to your group, making management easier.
- Network Interface Card (NIC): think of it like an ethernet port. It identifies the VM and connects it to the rest of the network.
- Network Security Group (NSG): allows for certain things to go in and out of the VM.

### Overview

- OS (image). there are 253 images that exist in the Azure marketplace.
- chip architecture (Arm64, x64, etc.)
- VM Size:
  - vCPUs
  - RAM
  - disk size
- Networking, (virtual network, inbound/outbound IP rules, etc.)
- VM Size Templates:
  - A Series: good low-cost option, good for dev/test, small web servers, small database, proof of concept, etc. they can only be used in Gen1 versions of the provided OS.
  - B Series: burstable workloads that do not need to work continuously.
  - D Series: good for production app servers and production databases. allows access to latestm AMD and Intel chipsets, SSDs.
  - E Series: good for memory intensive workloads; data analytics servers, large in-memory workloads. allows limits on vCPU count to save on cost.
  - M Series: used for the most memory intensive workloads.
  - F Series: compute intensive workloads; network appliances, batch processes, video encoding and rendering, etc.
  - L Series: storage optimized workloads; data warehouses, large transactional databases.
  - N Series: GPU Optimized workloads; graphic-intensive applications, gaming applications
  - H Series: highest compute workloads; massive computation algorithms. typically not used for app dev.
- Azure Disk Storage:
  - IOPS: input/output operations per second
  - throughput: the size of the requests
  - latency

### Create a Virtual Machine Resource

#### Basics

- availability zones:
  - "No infrastructure redundancy required": your VM will be deployed on a single machine. No backup if that machine goes down.
  - "Availability zone": replicates the VM within the same region as it is contained in.
  - "Availability set": replicates the VM across different fault domains. a fault domain is a different machine within the same data center.
  - "Virtual machine scale set": distribute across zones and fault domains at scale.
- "Run with Azure Spot discount":
  - allows a discount for unused resources. it can suddenly turn off your VM if paying customers buy up all available resource. usually best to leave this unchecked for consistency.
- Administrator account:
  - create an admin account for your VM.
- "select inbound ports": selectively choose access points for security. leaving RDP checked and the rest unchecked is fine for testing.

#### Disks

- "Encryption at host": encrypts the drive at the host level.
- "Delete with VM": deletes the disk resource if the VM gets deleted.
- "Key management": allows for creation of keys for "ultra disks"
- "Data disks for vm-...":
  - allows for additional drives to attach to VM.
- Networking
- "Virtual network": a VNET resource necessary for the VM.
- "Public IP": necessary to be accessed by the outside world.
- "NIC network security group": create rules around what can and cannot be accessed.
- the ports section carries over from the basics page.
- "Delete public IP and NIC when VM is deleted":
  - does as the name suggests, you should check the box.
- "Enable accelerated networking":
  - gives you better throughput for a slightly higher cost. recommended to leave it on.
- Load balancing:
  - process of having multipe machines share the resource cost of processes so no one machine becomes overloaded.

#### Management

- "Identity":
  - VM is assigned an identity by Azure so other services can see it, and so that you do not have to log in with the credentials every time you want another Azure service to utilize it.
- "Azure AD"
  - utilize Azure Active Directory credentials to log in.
- "Auto=shutdown":
  - highly recommended to turn on
  - creates a schedule for automatically spinning down your VM at a certain time, saving you a ton of money.
- "Site Recovery"
  - allows you to set up a disaster recovery plan. if your VM is hosting anything for production, this should be set up.
  
#### Monitoring

- Alerts:
  - allows you to set rules for certain VM conditions, e.i. email you when the storage is running low.

#### Create the VM

  - once created, go to the "Connect" section on the left. Download the RDP file and use RDP on Windows to connect to the VM.

## Azure Functions

- serverless
- resources on demand (pay for what you use)
- caveats:
  - stateless
  - short-lived (functions can time out if data takes too long to retrieve, etc.)
  - cold start: servers are not always on, and sometimes have to spin up once needed.
- durable functions:
  - make functions stateful
  - are longer running than regular functions

### Plans:

- consumption plan
  - the default plan with the caveats listed above.
- premium plan
  - mitigates the cold start issue by always keeping the server on, however this costs more.
- dedicated plan
  - hosts your azure function in an app service plan. it functions like any other app service and needs to be configured as such. it loses advantages of serverless but sometimes is needed over durable functions.

### Triggers and Bindings

- triggers
  - trigger a function. each function requires at least one. functions are essentially event handlers.
  - example triggers:
    - HTTP request (API)
    - timer
    - new item added to blob storage
    - new item is added to am event hub or queue
- bindings
  - a method of getting data in and out of a function.
  - bindings allow for connections to other resources. they can be input or output bindings.
- example use case: say you have a timer trigger every hour. the azure function listens to an *input binding* to a blob storage resource that tells the function how many new images were added in the last hour. the azure function uses an *output binding* to forward this information to a notification hub.


#### Security

- authorization levels: a persons level of access to the function
  - function: user only has access to the one function (as in the resource, not literally one function in code)
  - host: universal access to everything
  - anonymous: a function set up at this level allows access to everyone
- authorization keys:
  - function: access to a single function (again, the resource, not code)
  - admin: gives access to everything, and let's you run administrative routines on your functions.
  - system: this key gets generated for you, and is used by third party services to access your functions. this would occur when your function has inputs/outputs to/from other azure services.

#### Security: App Keys

In a created Function App resource page on Azure, there is an "App keys" tab on the left menu. There are two key types available:
- System keys
  - Created for you by extensions, e.g. when setting up a blob storage trigger.
  - Adds an extra layer of security for outside Azure services accessing your functions
- Host keys
  - Allow clients to authenticate to any function in your function app service.
  - Given one of these by default, called "default"
  - As well as a "master" key, which provides administrative access

## Static Web Apps

Static Web App workflow:
  - Host code on a repo
  - Use CI/CD to deploy:
    - Static Content (frontend)
    - API

3 Configuration types:
  1. Application configuration:
    - Staticwebapp.config.json
      - This file allows you to configure routing, authentication, authorization, feedback rules, HTTP response overrides, global HTTP header definitions, custom mime types, and networking.
  2. Build configuration (CI/CD)
  3. Application Settings

API component of static web apps:
- Features:
  - Integrated security
  - Seamless routing
  - Bring your own API
- Constraints:
  - Only one API at a time
  - Route constraints
  - Security constraints
  - Fixed duration (total of 45 seconds)
  - Only HTTP requests supported (no web sockets)
  - Network-isolated backends not supported

## Containers (e.g. Docker)

### Azure Container Instances

Not meant for large-scale applications.

### Azure Container Apps

- Get the power of Kubernetes without the headache of setting it up
- Autoscaling capabilities (*is* meant for large-scale applications)
- Best for:
  - Deploying API Endpoints
  - Hosting background processing apps
  - Handling event-driven processes
  - Running microservices

## Azure Kubernetes Service (AKS)

- Kubernetes makes managing containers easier

## Microservices

- Microservices are a way of breaking up a monolithic application into smaller, more manageable pieces.
- Internet -> API Gateway -> Microservices -> Event Grid
- Microservices are:
  - small, modular pieces
  - loosely coupled
  - scale efficiently
  - allow work to be easily divided
  - polyglot capable (can be written in different languages)
  - isolates problems

### Azure Service Fabric

- Simplifies the process of building microservices
- Provides tools and services:
  - programming models
  - service templates
  - deployment tools
  - service integration (Azure DevOps, Kubernetes Service, Azure Monitor, etc.)
- Support for multiple languages: (.NET, Java, C++, Node.js, Python, etc.)
- Allows for automatic scaling, rolling updates, and fault tolerant design patterns

## Data Storage

- Relational Database Management System (RDBMS)
- NoSQL
- File-based storage (txt, csv, excel, binary, etc.)

### Azure SQL Database

Advantages:
- Relational Db
- Provides high availability and disaster recovery
- Supports hybrid scenarios with on-prem SQL Server instances

Disadvantages:
- Higher cost compared to other services
- Not the best for non-relational data

### Azure Blob Storage

Scalable object storage for unstructured data.

Advantages:
- Low cost
- Supports multiple access tiers:
  - Hot: frequently accessed data (most expensive)
  - Cool: infrequently accessed data
  - Archive: rarely accessed data (least expensive)
- Geo-redundant storage (replicated data between multiple data centers)

Disadvantages:
- Not designed for high-performance transactional data (such as data in a relational database)

### Azure Table Storage

NoSQL key-value store.

Advantages:
- Low cost
- Automatic indexing of data for faster queries
- Scalable for high-traffic workloads

Disadvantages:
- Limited query capabilities
- May not be suitable for complex data models or hierarchical data

### Azure Cosmos DB

Supports multiple data types including NoSQL and unstructured data.

Advantages:
- Multi-model database (NoSQL, key-value, graph, etc.)
- Global distribution
- Provides guaranteed low-latency and high-throughput access to data

Disadvantages:
- Higher cost compared to other services
- Steeper learning curve

### Azure Data Lake Storage

- Supports high-volume analytics
- Can handle unstructured, semi-structured, and structured data

Advantages:
- Scalable and secure
- Made for big data
- Provides fine-grained access control for data protection

Disadvantages:
- May require specialized skills to manage and configure for optimal performance

### Create a SQL Database

SQL Deployment Options:
- SQL Database: Cloud based
- SQL Managed Instance: Hybrid cloud, "lift and shift" migration
- SQL Virtual Machine: IaaS, SQL Server VM to move on-prem data onto

SQL Elastic Pool:
- Allows you to share the same resources between multiple databases at the cost of performance

Service and compute tiers:
VCore: 
- Infrastructure oriented; X number of cores, Y amount of memory
DTU (Database Throughput Unit):
- "More encompassing", based around a Microsoft standard. Basically vBucks.
Provisioned:
- pre-allocated resources, billed per hour based on vCores configured
Serverless:
- auto-scales based on workload, billed per second based on vCores used

Connectivity method:
- Public endpoint: allows access from anywhere outside of Azure
- Private endpoint: allows access from a specific VNET

Firewall Rules:
- Allow Azure services and resources to access this server: Should be set to YES
  - Makes it so other Azure services can read from your database
- Add current client IP address: Should be set to YES
  - Adds your current IP address to the firewall whitelist

Connection policy: can leave as default

Security is "outside the scope of this course"

Data source: no data, backup from storage, or sample data (AdventureWorksLT adds dummy data)

After created:
- your database will have a "Connection strings" link.
- your database will have a "Server name" link, used to access the database from outside of Azure (remember that you need to add your IP address to the firewall whitelist up in the "Firewall Rules" section)
  - You can use this server name with a tool like SQL Server Management Studio:
    - Server name: [Server name].database.windows.net
    - Authentication: Azure Active Directory - Universal with MFA
      - (requires your Azure subscription be set up with MFA)

## Advanced Application Enhancements

### Azure App Configuration
- Feature flags
- Dynamic configuration: allows you to change configuration settings without having to redeploy your application
- Centralized configuration: allows you to manage configuration settings for multiple applications in a single place
- Azure Key Vault integration: allows you to store secrets in Key Vault and access them from App Configuration

### Azure Key Vault
- Keys: used for encryption
- Secrets:
  - Manual: Key/value pairs. used for passwords, connection strings, etc.
  - Certificate upload (old way)
- Certificates (new, modern way)
  - generates certificates

### Azure Cache for Redis
- (Called "Redis Cache" in Azure)
- Improves performance by caching database return values
- High availability so that if a cache node goes down, the cache is still available
- Designed with security and compliance in mind

### Azure Content Delivery Network (CDN)
- Globally distributed network that caches and delivers files such as images, videos, scripts, and other static or dynamic files with high performance and low latency
- If Redis cache is all about data, Azure CDN is all about files
- How it works: 
  - CDN caches files at "edge nodes" in different locations around the world
  - When a user requests a file, the CDN delivers the file from the edge node that is closest to the user
  - If the file is not cached at the edge node, the CDN retrieves the file from the origin server (e.g. Azure Blob Storage) and caches it at the edge node for future requests
- You can specify cache expiration policies, so that files automatically refresh say every 24 hours

## Misc

### Azure Monitor

- Azure Monitor
  - Monitoring service that shows performance and health of Azure resources and applications
  - Can monitor logs, trace requests, etc.
  - Systems that can integrate:
    - Event Hubs
    - Logic Apps
    - APIs
    - Hosted Partners (e.g. ElasticSearch)

### Azure API Management

Service that allows users to create, publish, manage, secure, and analyze APIs.

Developer portal:
  - autogenerated documentation and definitions
  - API access account creation and subscription
  - API testing
  - API analytics
API Gateway:
    - Is inserted between the client requests and the backend services
    - Security layer: verifies client key's are genuine
    - Throttles traffic based on quotas
    - Caching (if API is called over and over with same parameters, response will be cached)
    - Allows you to host multiple versions of the same API by hosting multiple versions under a single domain
    - Groups:
      - Admin
      - Developers
      - Guests
    - Open products can be used by anyone
    - Protected products require a subscription key

## Application Gateways

An application gateway is a web traffic load balancer and application delivery controller that manages traffic to your web applications.

Load balancing:
  - Distributes traffic to multiple servers
  - Improves performance, scalability, and availability
- WAF (Web Application Firewall)
  - Protects web applications from common web vulnerabilities and attacks
  - Rule sets (OWASP and other security standards, as well as application specific rules)
  - Rate limiting (prevents DDoS attacks)
  - Geo-filtering (blocks traffic from specific countries)
  - Real-time monitoring and logging
- Routing and request features
  - URL-based routing
  - SSL termination (encrypts and decrypts data for you)
  - HTTP header manipulation (adds, removes, or updates HTTP headers)
- Misc features
  - Session affinity: create a session variable used between calls that allows for the session to have state
  - Health probes: checks the health of the backend servers
  - Logging and diagnostics

## Azure in the IDE

Publishing an Azure API App service using Visual Studio:
- Click Build > Publish [Project Name]
- Click Azure, then Next
- Select Azure App Service
- Log in if not already
- Select your subscription/resource group
  - If you don't have one, click "Create a new instance" and follow the prompts
  - If you had a plan that offered deployment slots, you could select them (not necessary)
- Select "skip this step" if you do not need an API Management Service
- Click Finish
  - This creates an XML document with the publish settings so you don't have to go back and do all that again
- Click the publish button on the top right
- If running an API, make sure the swagger page generation isn't gated behind if(IsDevelopment())
- Rebuild
- Republish by clicking Build > Publish [Project Name]
  - (this should remember your publish settings from before)
  - Click the publish button on the top right

Visual Studio Code also has extensions that very easily let you deploy different services to Azure.