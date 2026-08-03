# AWS Multi-Region Web Application with Global Accelerator
A highly available web application deployed across two AWS Regions using AWS Global Accelerator, Application Load Balancers, and Amazon EC2 to provide automatic regional failover and improved availability.

## Scenario
A global learning platform hosts a simple public web application that users access daily. Right now, this application runs in only one AWS Region, behind a single Application Load Balancer and one EC2 instance.

## Challenges:

- Users in other continents face higher latency
- If the Region goes down, the entire application becomes unavailable
- There is no automatic failover or backup Region
- All traffic depends on one endpoint
- The team cannot direct users to the closest Region for better performance

As the platform grows internationally, the engineering team wants a multi-region architecture that provides global performance and seamless continuity during outages.

## Solution

I will deploy the same lightweight HTML web application in **two AWS Regions,**one primary and one secondary. Each Region will run its own EC2 instance behind an Application Load Balancer.

Then, using AWS Global Accelerator, we will create a single global endpoint that routes users to the nearest healthy AWS Region for better performance.

## Architecture
<img width="978" height="698" alt="image" src="https://github.com/user-attachments/assets/a08a7242-a2b8-401d-9b7f-5fcb54f187fa" />

1. Users access the application using the Global Accelerator DNS or static IP.
2. AWS Global Accelerator receives the request at the nearest AWS edge location.
3. Global Accelerator forwards traffic to the Primary Region endpoint group.
4. The Primary Region ALB routes the request to the EC2 instance hosting the application.
5. Route 53 Health Checks continuously monitor the Primary ALB for availability.
6. If the Primary Region becomes unhealthy, Global Accelerator automatically shirts traffic to the Secondary Region.
7. The Secondary Region ALB forwards traffic to its EC2 instance, which serves the application during failover.

## What This Project Demonstrates
By the end of this project I would've demonstrated:

- Deploying a styled web app in two AWS Regions
- Creating ALBs to expose the app in each Region
- Add Route 53 Health Checks for continuous monitoring
- Configuring AWS Global Accelerator with two endpoint groups
- Testing how traffic moves between Regions
- Simulating a failure and observe automatic failover
   
## Technologies Used
- AWS Global Accelerator - Global routing & failover
- Elastic Load Balancing (ALB) - Regional traffic distribution
- Amazon EC2 - Web application hosting
- Amazon VPC - Networking, subnets, routing
- Amazon Route 53 Health Checks - Regional availability monitoring
  
## ⭐ Deployment Guide
Each guide includes AWS console screenshots, configuration details, and validation steps.

1. [Build Primary Region](docs/01-build-primary-region.md)
2. [Build Secondary Region](docs/02-build-secondary-region.md)
3. [Configure Global Accelerator](docs/03-configure-global-accelerator.md)
4. [Failover Testing](docs/04-failover-testing.md)
5. [Cleanup](docs/05-cleanup.md)
