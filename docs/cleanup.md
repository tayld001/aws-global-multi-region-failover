Cleanup
To avoid ongoing charges, delete the following resources in both Regions:

Global Accelerator

1. Open Global Accelerator Console
2. Disable and delete the accelerator
3. Wait for the status to be Deleted

Route 53
 - Delete the two Health Checks
   
Load Balancers
 - Delete alb-region-a
 - Delete alb-region-b

Target Groups
 - Delete tg-region-a
 - Delete tg-region-b

EC2 Instances
 - Terminate web-server-a
 - Terminate web-server-b

Security Groups
 - Delete web-sg-a
 - Delete web-sg-b

Networking

Delete these items per Region in this sequence:

1. Remove route table associations → delete public route tables
2. Detach Internet Gateway → delete Internet Gateway
3. Delete subnets
4. Delete VPCs (multi-region-vpc-a, multi-region-vpc-b)
Once all resources are removed, charges will stop.
