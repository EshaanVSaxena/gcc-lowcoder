# Google Cloud Console-Lowcoder-Setup

## Creating a VPC Network
1. In the Google Cloud Console, in VPC Network, click on VPC Networks
    - Enable Compute Engine API
2. Create a VPC Network
    - Name the VPC Network
    - Disable IPv6
    - Create a custom subnet 
        - Name the subnet 
        - Choose the region best for your
        - Choose a IP Range
        - Private Google Access: On
        - Flow Logs: Off
        - In the subnet setting turn on Private Google Access (it is necessary to enable this setting otherwise the cloud run services will not be able to communicate with one another)
            - This will be in the Edit subnet section is using a custom subnet creation mode
            - If subnet creation mode is automatic then you will have to go to the region of your VPCnetwork and enable the Private Google Access setting from there once the VPCis created.

![VPC Network Setup.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/2cf8a390724968a9864b8c9967018d565502f89b/VPC%20Network%20Setup.JPG)

## Creating a Serverless VPC Access Connector
1. In the VPC Network, create a Connector
    - Enable serverless vpc access API
2. Create a Serverless VPC Connector
    - Name the Serverless VPC Connector
    - Choose the same region as the VPC Network
    - Network: Select the VPC Network that was previously created
    - Subnet:
        - Custom IP range
        - IP range (cannot be the same subnet that was created before) 

![Serverless VPC Access Connector Setup.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/8d4e5d17533415c6ec4ae800381be5836923b961/Serverless%20VPC%20Access%20Connector%20Setup.JPG)

##Creating a Cloud NAT
- Creating a Cloud NAT is necessary because when connecting the lowcoder services to MongoDB, you must whitelist an IP address which can access the MongoDB cluster.
    - Google Cloud services will use different IP addresses each time to communicate with the internet unless an Cloud NAT is set up on the VPC
1. Within the “Network Services” in the Google Cloud Console click on “Cloud NAT” and then “Create Cloud NAT gateway”
2. Name the Cloud NAT gateway
3. Set the NAT type to public
4. Select a Cloud Router 
    - Connect it to the VPC Network created previously
    - Select the region for your Router (the one your services and VPC are running on)
    - Create a New Router 
        - Name the router
5. Cloud NAT mapping 
    - Source: Primary and secondary ranges for all subnets
    - Cloud NAT IP Addresses: Manual
        - IP Addresses:
            - Premium Network Service Tier
            - IP address: 
                - Create and Name an IP address and then click on “Reserve”
            - IP draining: Off (default)
6. Click on Create Cloud NAT 
7. After creating an Cloud NAT IP address, whitelist this IP address on your MongoDB cluster

![Cloud NAT Setup.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/698541f903287406630f701a474c61d0916f097b/Cloud%20NAT%20Setup.JPG)

## Creating a Redis Instance
1. Within Google Cloud Console in Memorystore, in the redis page, create a redis instance
2. Enable Google Cloud Memorystore for Redis API
3. Create a Redis Instance:
    - Name Redis instance in the Instance ID
    - Basic Tier
    - 1 GB of capacity is more than sufficient in our environment however if you face trouble then you can always adjust the capacity as necessary
    - Choose the same region as the VPC Network
    - Set up Connection: connect your redis instance to the VPC network created previously

![Redis Instance Setup.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/8d4e5d17533415c6ec4ae800381be5836923b961/Redis%20Instance%20Setup.JPG)

## Setting up the Node Service
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-node-service
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-node-service)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to Internal (this is necessary to not expose the Node Service to the rest of the internet)
7. Authentication: Allow unauthenticated access invocations

![Node Service Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/Node%20Service%20Settings.JPG)

### Container Settings
1. Container port: 6060
2. CPU allocation:
    - Only allocated during the request processing
3. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
4. Execution environment: Default

![Node Service Container Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/Node%20Service%20Container%20Settings.JPG)

5. Environment Variable:
    - Name 1: LOWCODER_API_SERVICE_URL
    - Value 1: Paste the Api Service URL (Once the API Service is created after the next step

![Node Service Environment Variables.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/d578f898c8884e8c31cb1b1468664439d4d84b24/Node%20Service%20Environment%20Variables.JPG)

### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route only requests to private IPs to the VPC (this is necessary to not expose the Node Service to the rest of the internet)

![Node Service Networking Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/816341197861acb84a3f41762d67059e6c1030c0/Node%20Service%20Networking%20Settings.JPG)

## Setting up the API Service
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-api-service
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-api-service)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to Internal (this is necessary to not expose the API Service to the rest of the internet)
7. Authentication: Allow unauthenticated access invocations

![API Service Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/API%20Service%20Settings.JPG)

### Container Settings
1. Container port: 8080
2. Capacity (allocate as necessary) 
    - Memory: 1 GiB 
    - CPU: 1
3. Execution environment: Second generation

![API Service Container Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/a0c34bf799035819f04f48ef884f4200c973e3e9/API%20Service%20Container%20Settings.JPG)

4. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
    - Variable 1: 
      - Name 1: REDIS_URL
      - Value 1: use: redis://10.0.0.0:6379?db=databasename and replace 10.0.0.0 with your redis instance ip address and database name with the name of your database
    - Variable 2: 
      - Name 2: MONGODB_URL
      - Value 2: Paste the MongoDB URL
    - Variable 3: 
      - Name 3: LOWCODER_NODE_SERVICE_URL
      - Value 3: Paste the Node Service URL
    - Variable 4: 
      - Name 4: ENABLE_USER_SIGN_UP
      - Value 4: TRUE
            - If it is a new set up then set it to true, otherwise if it will be used for a existing set up then set it to FALSE
    - Variable 5: 
      - Name 5: ENCRYPTION_PASSWORD
      - Value 5: lowcoder.org
    - Variable 6: 
      - Name 6: ENCRYPTION_SALT
      - Value 6: lowcoder.org
    - Variable 7: 
      - Name 7: CORS_ALLOWED_DOMAINS
      - Value 7: *

![API Service Envirnoment Variables.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/c5d3f5bb8824de49807e27cbb31336441dd785f1/API%20Service%20Envirnoment%20Variables.JPG)

### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route all traffic to the VPC

![API Service Networking Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/816341197861acb84a3f41762d67059e6c1030c0/API%20Service%20Networking%20Settings.JPG)

## Setting up the Front-End Service 
1. In the Google Console go to Cloud Run
2. Create a Service 
3. In the Container Image URL input: lowcoderorg/lowcoder-ce-frontend
    - This is from the lowcoder docker hub (https://hub.docker.com/r/lowcoderorg/lowcoder-ce-frontend)
4. Select the Region, same as the region for the VPC Network
5. CPU allocation and pricing: CPU is only allocated during request processing
6. Ingress Control is set to all (the front-end should be exposed to the internet)
7. Authentication: Allow unauthenticated access invocations

![Frontend Service Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/Frontend%20Service%20Settings.JPG)

### Container Settings 
1. Container port: 3000
2. Capacity (allocate as necessary) 
    - Memory: 512 MiB 
    - CPU: 1
3. Execution environment: Default

![Frontend Service Container Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/Frontend%20Service%20Container%20Settings.JPG)

4. Environment Variable: add any other environment variable as per your requirement (list of environment variables https://raw.githubusercontent.com/lowcoder-org/lowcoder/main/deploy/docker/docker-compose-multi.yaml)
    - Variable 1: 
      - Name 1: LOWCODER_API_SERVICE_URL
      - Value 1: Paste the Api Service URL
    - Variable 2: 
      - Name 2: LOWCODER_NODE_SERVICE_URL
      - Value 2: Paste the Node Service URL

![Frontend Service Environment Variables.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/4df71f618e95330d771f811540905c551b9a70ab/Frontend%20Service%20Environment%20Variables.JPG)

### Networking Settings
- Connect to a VPC for outbound traffic
    - Use Serverless VPC Access Connectors
       - Network: Select the VPC network that was created 
    - Traffic routing:
      - Route all traffic to the VPC

![Frontend Service Networking Settings.JPG](https://github.com/lmt-ventures/GCC-Lowcoder-Setup/blob/816341197861acb84a3f41762d67059e6c1030c0/Frontend%20Service%20Networking%20Settings.JPG)

### Post Deployment Settings
- Can set up a DNS through Google Domains 

