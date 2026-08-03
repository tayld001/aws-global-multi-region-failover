In this section, I will observe how AWS Global Accelerator automatically redirects traffic from the Primary Region (Region A) to the Secondary Region (Region B). This is the final part of the project, demonstrating that the multi-region setup is effective.

Make sure the Global Accelerator is fully deployed, both endpoints show Region A 100% and Region B 0%, both ALBs are healthy and have copied the Global Accelerator DNS name.

# Step 1: Access Your App Through Global Accelerator
 - Copy the Global Accelerator DNS name
 - Paste it in the browser
 - The Region A version would appear
<img width="975" height="357" alt="image" src="https://github.com/user-attachments/assets/891c7276-70eb-4057-af40-975e66e865fc" />

# Step 2: Simulate a Failure in Region A
This is the simplest way to simulate failure is to temporarily stop the instance service Region A.
 - Go to the EC2 Console in your Primary Region
 - Select web-server-a
 - Click Instance state → Stop
<img width="975" height="181" alt="image" src="https://github.com/user-attachments/assets/a6d3625c-531b-4659-b8e1-5ca57b7facf7" />

 - Once its stopped the ALB in Region A will begin failing health checks.
<img width="975" height="122" alt="image" src="https://github.com/user-attachments/assets/13fc7c32-f055-4ee9-9d10-3512fc84cd7f" />

 - The Global Accelerator health check will also show Region A as Unhealthy
<img width="975" height="120" alt="image" src="https://github.com/user-attachments/assets/c7533f90-e6e9-4349-bbe0-f64e8ac8c7f5" />

This may take around 1-2 minutes depending on health check intervals.

# Step 3: Refresh Your Browser
 - Refresh the Global Accelerator URL
 - The Region B version would appear
<img width="975" height="346" alt="image" src="https://github.com/user-attachments/assets/b0af3f59-c662-4917-8997-30ac6ec4986a" />

# Step 4: Restore Region A
 - Start the EC2 instance again.
<img width="975" height="185" alt="image" src="https://github.com/user-attachments/assets/c6fb7a3b-9dcf-40e7-b947-dfedb1254964" />

 - Wait for:
   - EC2 → Running
   - ALB health check → Healthy
   - Global Accelerator → Region A endpoint shows Healthy
   - Once Region A is healthy again, refresh the DNS name.
<img width="975" height="348" alt="image" src="https://github.com/user-attachments/assets/6b67d4c1-2888-469c-ad86-3226e2547dd2" />


# Closing Thoughts
In this project, I built a fully working multi-region web application using AWS Global Accelerator.

Deploying identical applications in two AWS Regions, exposing them through ALBs, set up health checks, and connected everything using a single global endpoint.

This is a production-grade pattern used by global platforms to achieve:

 - High availability
 - Low latency for worldwide users
 - Resilience against Regional outages
Now I understand how to design, deploy, and test a multi-region architecture, which is an essential skill for Cloud Engineers.
