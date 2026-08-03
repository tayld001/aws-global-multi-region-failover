## Overview of Project

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

## About the Project
In this hands-on project, I will:

- Deploy a styled web app in two AWS Regions
- Create ALBs to expose the app in each Region
- Add Route 53 Health Checks for continuous monitoring
- Configure AWS Global Accelerator with two endpoint groups
- Test how traffic moves between Regions
- Simulate a failure and observe automatic failover
  
By the end, I will have a fully operational multi-region architecture.

## Technologies Used
AWS Global Accelerator - Global routing & failover
Elastic Load Balancing (ALB) - Regional traffic distribution
Amazon EC2 - Web application hosting
Amazon VPC - Networking, subnets, routing
Amazon Route 53 Health Checks - Regional availability monitoring
