# AWS-Global-Multi-Region-Failover
I am deploying the same lightweight HTML web application in **two AWS Regions,**one primary and one secondary. Each Region will run its own EC2 instance behind an Application Load Balancer.  Then, using AWS Global Accelerator, I will create a single global endpoint that routes users to the nearest healthy AWS Region for better performance.
