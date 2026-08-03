In this section, I will deploy the first version of the web application in your Primary Region (for example, us-east-2). This includes creating the networking, launching the EC2 instance, and exposing it through an Application Load Balancer.

Before beginning: Select the Primary Region

At the very top-right of the AWS Console:

1. Click the region dropdown
- Select your Primary Region.
Everything you create in the next steps will now belong to this region.

Note: Choose any AWS Region for the Primary Region, but once selected, make sure to use the same region consistently for all resources in this part of the project.

# Step 1: Create the VPC
- Open the VPC Console → Create VPC
- Choose VPC Only.
- Name it: multi-region-vpc-a
- CIDR block: 10.10.0.0/16
<img width="975" height="344" alt="image" src="https://github.com/user-attachments/assets/d2c53666-013e-4c75-82ef-7a2bfa80068d" />

 - Create the VPC.

# Step 2: Create Two Public Subnets
Now that the VPC is created, the next step is to add two public subnets inside it.

Public subnets are needed because:

- The Application Load Balancer must exist in at least two Availability Zones
- Our EC2 instance should be able to receive traffic from the internet
- We will create both subnets inside the VPC multi-region-vpc-a.

## Create Subnet 1

 - Go to VPC Console → Subnets → Create Subnet
 - Select VPC: multi-region-vpc-a ← (important!)
 - Enter the details:
    - Name: subnet-a1
    - Availability Zone: any AZ (e.g., us-east-2a)
    - IPv4 Subnet CIDR block: 10.10.1.0/24
<img width="975" height="459" alt="image" src="https://github.com/user-attachments/assets/69c3ac1c-0493-47b1-a2c6-d6f13ba312de" />

   - Create the subnet
   - After creation, select the subnet → Actions → Edit subnet settings
   - Turn on: Auto-assign IPv4 public address
<img width="975" height="435" alt="image" src="https://github.com/user-attachments/assets/66339d9d-3b9a-4416-8a41-ad5696d35c87" />

## Create Subnet 2

 - Click Create Subnet again
 - Select VPC: multi-region-vpc-a
 - Enter the details:
    - Name: subnet-a2
    - Availability Zone:
    - IPv4 CIDR block: 10.10.2.0/24
<img width="975" height="477" alt="image" src="https://github.com/user-attachments/assets/7f4b5c05-e6f2-4f5d-876c-86b22ac0b5e2" />

 - Create the subnet
 - Enable: Auto-assign IPv4 public address
<img width="975" height="458" alt="image" src="https://github.com/user-attachments/assets/6ff35212-eb30-4093-b77d-aa7e9dcb055c" />


Without the auto-assign Public IP the EC2 instance will not get a public IP which means I can't test it directly through the internet and the ALB can't reach it.

# Step 3: Attach an Internet Gateway
The VPC needs an Internet Gateway (IGW) to allow outbound traffic and let users reach the application from the internet.

 - In the VPC console, go to Internet Gateways → Create internet gateway
 - Name tag: igw-region-a
 - Click Create internet gateway
<img width="975" height="311" alt="image" src="https://github.com/user-attachments/assets/eb0249b9-4f0a-43cc-a1b8-0d5a7ee0e9b1" />

 - Select the IGW → Actions → Attach to VPC
 - Choose multi-region-vpc-a
<img width="975" height="203" alt="image" src="https://github.com/user-attachments/assets/f12b9033-0cba-44e9-ba44-ea61cfd791e1" />

Once the VPC is attached, I am capable of sending and receiving traffic from the internet.

# Step 4: Create a Public Route Table
To make the subnets truly public, they have to use a route table that'll send all internet traffic through the Internet Gateway.

 - In the VPC console, go to Route Tables → Create route table
 - Name it: public-rt-a
 - Select VPC: multi-region-vpc-a
 - Click Create route table
<img width="975" height="320" alt="image" src="https://github.com/user-attachments/assets/05720235-f651-4c46-97a7-306e3972bc76" />

Add the Internet Route
 - Open the route table that was just created
 - Go to the Routes tab → Edit routes
 - Add:
    - Destination: 0.0.0.0/0
    - Target: your Internet Gateway (igw-region-a)
<img width="975" height="223" alt="image" src="https://github.com/user-attachments/assets/5f548d52-ccc8-4f55-a22f-130938b80079" />

 - Save the route
This ensures the traffic from the subnet can reach the internet.

Associate Both Subnets
 - Go to the Subnet associations tab → Edit subnet associations
 - Select:
    - subnet-a1
    - subnet-a2
<img width="975" height="250" alt="image" src="https://github.com/user-attachments/assets/1d2037e9-0b51-438e-8aa5-1eabf90e8cea" />

 - Save associations
Now these two subnets are officially public subnets which means that the resources inside can be accessed from the internet via the IGW.

# Step 5: Create the Security Group
The security group will be shared by the EC2 instance and the ALB.

 - In the VPC console, go to Security Groups → Create security group
 - Name: web-sg-a
 - Description: Allow anyone on the internet to access the web application.
 - Select VPC: multi-region-vpc-a
 - Add an inbound rule
    - Type: HTTP
    - Port: 80
    - Source: 0.0.0.0/0
<img width="975" height="444" alt="image" src="https://github.com/user-attachments/assets/c7565d65-071e-48d8-8d22-5a0851e4ee67" />

Outbound rules can be left as the default "Allow All," that lets the instance download packages. Also, this allows anyone on the internet to access the web application.

# Step 6: Launch the EC2 Instance (Region A Version)
This EC2 instance will host the Region A version of your web application.

 - Go to the EC2 Console → Launch Instance
 - Name the instance: web-server-a
 - Choose AMI: Amazon Linux 2023
<img width="975" height="594" alt="image" src="https://github.com/user-attachments/assets/a9fc62dd-6b5e-4ac5-9a09-58585bef8d8d" />

 - Choose instance type: t3.micro (Free Tier eligible)
 - Configure Networking
    - VPC: multi-region-vpc-a
    - Subnet: subnet-a1 (one of my public subnets)
    - Security Group: web-sg-a
<img width="975" height="694" alt="image" src="https://github.com/user-attachments/assets/f8843118-8b8a-4f95-876f-944357d9050b" />

 - Add User Data (Paste entire HTML file here)
 - Scroll to Advanced details → User data
 - Paste the following script.

```
#!/bin/bash
yum install -y httpd
systemctl enable httpd
systemctl start httpd

cat <<'EOF' > /var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Multi-Region Web Application</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --primary: #1d4ed8;
      --primary-soft: #dbeafe;
      --bg-dark: #020617;
      --bg-darker: #0b1220;
      --text-main: #0f172a;
      --text-muted: #64748b;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: radial-gradient(circle at top, var(--bg-darker), var(--bg-dark));
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: var(--text-main);
    }
    .shell { width: 100%; max-width: 900px; padding: 24px; }
    .card {
      background: #f8fafc;
      border-radius: 24px;
      box-shadow: 0 20px 45px rgba(15, 23, 42, 0.45),
                  0 0 0 1px rgba(148, 163, 184, 0.25);
      overflow: hidden;
    }
    .card-header {
      background: var(--primary);
      color: #e5f0ff;
      padding: 18px 28px;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .title { font-size: 20px; font-weight: 600; }
    .tag {
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.12em;
      padding: 6px 10px;
      border-radius: 999px;
      border: 1px solid rgba(219, 234, 254, 0.35);
      background: rgba(15, 23, 42, 0.3);
    }
    .card-body { padding: 40px 32px 30px; text-align: center; }
    .region-label {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      padding: 6px 14px;
      border-radius: 999px;
      background: var(--primary-soft);
      color: #1e3a8a;
      font-size: 13px;
      font-weight: 500;
    }
    .dot {
      width: 8px;
      height: 8px;
      border-radius: 999px;
      background: #22c55e;
      box-shadow: 0 0 0 6px rgba(34, 197, 94, 0.25);
    }
    h1 {
      margin: 26px 0 10px;
      font-size: clamp(28px, 4vw, 38px);
      color: #0f172a;
    }
    .region-name { color: var(--primary); }
    .subtitle {
      margin: 6px 0 18px;
      font-size: 15px;
      color: var(--text-muted);
    }
    .meta-row {
      display: flex;
      justify-content: center;
      gap: 24px;
      margin-top: 10px;
      font-size: 14px;
      color: #4b5563;
      flex-wrap: wrap;
    }
    .meta-label {
      font-weight: 500;
      color: #111827;
    }
    .footer {
      padding: 16px 24px 20px;
      text-align: center;
      border-top: 1px solid #e2e8f0;
      font-size: 13px;
      color: #6b7280;
      background: #f9fafb;
    }
    .brand {
      font-weight: 600;
      color: #0f172a;
    }
  </style>
</head>
<body>
  <div class="shell">
    <div class="card">
      <div class="card-header">
        <div class="title">Multi-Region Web Application</div>
        <div class="tag">Global Accelerator • Active</div>
      </div>

      <div class="card-body">
        <div class="region-label">
          <span class="dot"></span>
          <span>Primary Region</span>
        </div>

        <h1>
          Served from:
          <span class="region-name">Region A (us-east-2)</span>
        </h1>

        <p class="subtitle">
          Traffic is currently landing in the <strong>Ohio</strong> AWS Region
          through <strong>AWS Global Accelerator</strong>.
        </p>

        <div class="meta-row">
          <div>
            <span class="meta-label">Endpoint:</span>
            Application Load Balancer
          </div>
          <div>
            <span class="meta-label">Status:</span>
            Healthy
          </div>
        </div>
      </div>
    </div>
  </div>
</body>
</html>
EOF
```

<img width="975" height="456" alt="image" src="https://github.com/user-attachments/assets/f5df4e69-4929-4f87-a50d-4e6141c2c04c" />

 - Launch the Instance.

# Step 7: Create the Target Group
The target group defines where the ALB should send traffic.

 - In the EC2 console, go to Target Groups → Create target group
 - Target type: Instance (because we want the ALB to forward traffic to an EC2 instance)
 - Name: tg-region-a
 - Protocol & Port: HTTP, port 80
 - VPC: Ensure it shows multi-region-vpc-a
<img width="975" height="478" alt="image" src="https://github.com/user-attachments/assets/60a4b0da-c411-42f9-a9ea-2098d1c8404c" />

 - Click Next and register your EC2 Instance.
 - From the list of instances, select web-server-a
 - Click Include as pending
<img width="975" height="436" alt="image" src="https://github.com/user-attachments/assets/b39a6ad6-e69b-4f98-9aad-7a54d68b1132" />

 - Click Next then Create target group

# Step 8: Create the Application Load Balancer (ALB)
The ALB will act as the public entry point for all traffic coming into Region A.

 - Open the EC2 Console → Load Balancers
 - Click Create Load Balancer
 - Select Application Load Balancer
 - Enter the name: alb-region-a
 - Scheme: Internet-facing (this makes the ALB reachable from the internet)
<img width="975" height="393" alt="image" src="https://github.com/user-attachments/assets/de815821-5c6b-409f-8df8-153245af4fed" />

 - VPC: Select multi-region-vpc-a
 - Under Mappings, choose both public subnets:
    - subnet-a1
    - subnet-a2
 - Under Security Groups: Select web-sg-a
<img width="975" height="357" alt="image" src="https://github.com/user-attachments/assets/fb3393f0-c5cf-4ef9-ba3e-18782fa0757b" />

 - Target group: tg-region-a
<img width="975" height="460" alt="image" src="https://github.com/user-attachments/assets/42c1dcc3-2488-47c6-96f0-db072ea35882" />

 - Create load balancer

# Step 9: Verify Everything Works
Once the ALB status becomes Active:

 - Open the EC2 Console → Load Balancers
 - Select alb-region-a
 - Copy the DNS name from the details panel
 - Paste it into your browser

Now the Region A web application is being shown:
<img width="975" height="367" alt="image" src="https://github.com/user-attachments/assets/ccdd5078-ec3f-46b5-aa2f-f67af07676c4" />





