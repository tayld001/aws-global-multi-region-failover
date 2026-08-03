In this section, I will:

 - Create Route 53 Health Checks for both Regional ALBs
 - Configure AWS Global Accelerator to use these ALBs as endpoints
 - Build a single global entry point that automatically fails over between Regions
This completes the global resilience setup of the multi-region application.

Both Route 53 Health Checks and AWS Global Accelerator are global services which means there's no need to switch AWS Regions when creating health checks or configuring Global Accelerator.

# Step 1: Create a Health Check for Region A
 - Open the Route 53 Console
 - In the left menu, click Health checks
 - Click Create health check
 - Name: hc-alb-region-a
 - Resource: Endpoint
 - Specify endpoint by: Domain name
 - Protocol: HTTP
 - Domain name: paste the DNS name of alb-region-a
<img width="975" height="344" alt="image" src="https://github.com/user-attachments/assets/683a51ec-beee-4741-a8ad-ff801ecb53ed" />

 - Leave the remaining settings at their defaults
 - Click create health check
If the ALB and EC2 instance are reachable it should show Healthy.
<img width="975" height="188" alt="image" src="https://github.com/user-attachments/assets/084f6ada-4431-4c6c-8a30-7d72d05387b3" />

# Step 2: Create a Health Check for Region B
 - Click Create health check
 - Name: hc-alb-region-b
 - Resource: Endpoint
 - Specify endpoint by: Domain name
 - Protocol: HTTP
 - Domain name: paste the DNS name of alb-region-b
<img width="975" height="342" alt="image" src="https://github.com/user-attachments/assets/1862f67c-531a-4f0a-8f97-c65e22534c1d" />

 - Leave the remaining settings at their defaults
 - Click create health check
If the ALB and EC2 instance are reachable it should show Healthy.
<img width="975" height="191" alt="image" src="https://github.com/user-attachments/assets/13856751-d4e5-48b9-9f29-8f41025c6bf8" />

So far, Route 53 is now monitoring both Regions:
 - Region A ALB → Health Check A
 - Region B ALB → Health Check B

# Step 3: Create the Global Accelerator
 - Open the AWS Global Accelerator Console
 - Click Create accelerator
 - Name: multi-region-accelerator
 - Keep Accelerator type: Standard
<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/c9c849ac-8c4f-4982-a6df-4dc0f38c8b77" />

 - Click Next
 - Create the Listener
    - Port range: 80
    - Protocol: TCP
    - Client affinity: None
<img width="975" height="251" alt="image" src="https://github.com/user-attachments/assets/c9d9808e-00cb-453d-ac22-23b0c20197fa" />

 - Click Next
 - Add the Endpoint Group for Region A, this is the Primary Region.
    - Region: us-east-2 (or the region you have selected.)
    - Traffic dial: 100% (All traffic flows to Region A by default)
 - On the same “Add endpoint groups” page: Click Add endpoint group again
    - Region: us-west-2 (or the region you have selected.)
    - Traffic dial: 0

Now there are two endpoint groups, one for each Region.
<img width="975" height="427" alt="image" src="https://github.com/user-attachments/assets/927c3989-0a6f-42c4-af97-20707b3b6530" />

 - Click next to the add endpoints page

# Step 4: Add the Actual Endpoints (ALB)
Now it is time to add one endpoint (ALB) to each endpoint group:

For Endpoint Group: Region A
 - Click Add Endpoint
 - Endpoint type: Application Load Balancer
 - Endpoint: alb-region-a

For Endpoint Group: Region B
 - Click Add Endpoint
 - Endpoint type: Application Load Balancer
 - Endpoint: alb-region-b
<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/a2005a37-7453-422b-be6f-4a8a52a02208" />

 - Click create accelerator

Deployment takes about 2-3 minutes.

# Step 5: Copy the Accelerator DNS Name
Once deployed, there'll be:
 - Two static anycast IP addresses
 - One Global Accelerator DNS name
<img width="975" height="298" alt="image" src="https://github.com/user-attachments/assets/85c461a0-f708-4318-b798-ce1ffdf0de79" />

Save the DNS name for the failover testing. So now there's a single global entry point via AWS Global Accelerator with automatic health-based failover.
