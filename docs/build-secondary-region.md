In this section, I will deploy the same web application in the secondary region (us-west-2). This will provide regional redundancy, which means the app stays available even if the primary region becomes unavailable.The setup closely mirrors Region A, but there are minor changes such as naming and the Region B HTML file.

# Change Regions
At the top-right of the AWS Console:
 - Click the Region selector
 - Choose the Secondary Region. Example: Oregon - us-west-2
<img width="975" height="275" alt="image" src="https://github.com/user-attachments/assets/55f3d479-b6b8-43eb-ba02-17d2417fac74" />

All the resources that are created in this section will belong to the secondary region.

# Step 1: Create the VPC
- Open the VPC Console → Create VPC
- Choose VPC Only.
- Name it: multi-region-vpc-b
- CIDR block: 10.20.0.0/16
<img width="975" height="521" alt="image" src="https://github.com/user-attachments/assets/d5082dff-9d49-4841-a1fe-03249caf2626" />

 - Create the VPC.

# Step 2: Create Two Public Subnets
Just like Region A, two public subnets are needed.

## Create Subnet 1

 - Go to VPC Console → Subnets → Create Subnet
 - Select VPC: multi-region-vpc-b
 - Enter the details:
    - Name: subnet-b1
    - Availability Zone: any AZ (e.g., us-west-2a)
    - IPv4 Subnet CIDR block: 10.20.1.0/24
<img width="975" height="477" alt="image" src="https://github.com/user-attachments/assets/abe54722-b9dc-4971-853b-1aae41cce9f2" />

   - Create the subnet
   - After creation, select the subnet → Actions → Edit subnet settings
   - Turn on: Auto-assign IPv4 public address
<img width="975" height="432" alt="image" src="https://github.com/user-attachments/assets/1136ea2c-ce90-471d-aa9e-0b9460809a12" />

## Create Subnet 2

 - Click Create Subnet again
 - Select VPC: multi-region-vpc-b
 - Enter the details:
    - Name: subnet-b2
    - Availability Zone: us-west-2b
    - IPv4 CIDR block: 10.20.2.0/24
<img width="975" height="461" alt="image" src="https://github.com/user-attachments/assets/834cc148-a326-467e-ad51-2810cf1e7d0a" />

 - Create the subnet
 - Enable: Auto-assign IPv4 public address
<img width="975" height="431" alt="image" src="https://github.com/user-attachments/assets/c1e7ad65-bf13-4263-b7ab-47685f79beae" />

# Step 3: Attach an Internet Gateway
 - Go to Internet Gateways → Create internet gateway
 - Name tag: igw-region-b
 - Click Create internet gateway
<img width="975" height="308" alt="image" src="https://github.com/user-attachments/assets/1749f9b3-1122-4aca-975b-4b297e22400a" />

 - Select the IGW → Actions → Attach to VPC
 - Choose multi-region-vpc-b
<img width="975" height="199" alt="image" src="https://github.com/user-attachments/assets/ee3bbf0e-0683-4abe-b794-3ee3f949d807" />

Once the VPC is attached, I am capable of sending and receiving traffic from the internet.

# Step 4: Create a Public Route Table

 - In the VPC console, go to Route Tables → Create route table
 - Name it: public-rt-b
 - Select VPC: multi-region-vpc-b
 - Click Create route table
<img width="975" height="316" alt="image" src="https://github.com/user-attachments/assets/ff4685ca-5809-4026-a468-b69386600d35" />

Add the Internet Route
 - Open the route table that was just created
 - Go to the Routes tab → Edit routes
 - Add:
    - Destination: 0.0.0.0/0
    - Target: your Internet Gateway (igw-region-b)
<img width="975" height="216" alt="image" src="https://github.com/user-attachments/assets/e871baf7-22ec-4567-8873-8936f1e15e3a" />

 - Save the route
This ensures the traffic from the subnet can reach the internet.

Associate Both Subnets
 - Go to the Subnet associations tab → Edit subnet associations
 - Select:
    - subnet-b1
    - subnet-b2
<img width="975" height="249" alt="image" src="https://github.com/user-attachments/assets/faa3f588-2a22-4106-a402-e666172de2c6" />

 - Save associations
Now these two subnets are officially public subnets.

# Step 5: Create the Security Group

 - In the VPC console, go to Security Groups → Create security group
 - Name: web-sg-b
 - Description: Allow inbound HTTP traffic.
 - Select VPC: multi-region-vpc-a
 - Add an inbound rule
    - Type: HTTP
    - Port: 80
    - Source: 0.0.0.0/0
<img width="975" height="452" alt="image" src="https://github.com/user-attachments/assets/b2b874e9-96a9-4753-a760-7d60f5372502" />

This matches the primary region's configuration.

# Step 6: Launch the EC2 Instance (Region A Version)
This EC2 instance will host the Region B version of your web application.

 - Go to the EC2 Console → Launch Instance
 - Name the instance: web-server-b
 - Choose AMI: Amazon Linux 2023
<img width="975" height="679" alt="image" src="https://github.com/user-attachments/assets/53edb79e-cc98-46e3-9af9-aa9d79a22364" />

 - Choose instance type: t3.micro (Free Tier eligible)
 - Configure Networking
    - VPC: multi-region-vpc-b
    - Subnet: subnet-b1 (one of my public subnets)
    - Security Group: web-sg-b
<img width="975" height="527" alt="image" src="https://github.com/user-attachments/assets/668a9efb-ace5-4f9b-9388-c5be3f8ee821" />

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
      --primary: #059669;   /* Region B accent (green) */
      --primary-soft: #d1fae5;
      --bg-dark: #020617;
      --bg-darker: #0b1220;
      --text-main: #0f172a;
      --text-muted: #64748b;
    }

    * {
      box-sizing: border-box;
    }

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

    .shell {
      width: 100%;
      max-width: 900px;
      padding: 24px;
    }

    .card {
      background: #f8fafc;
      border-radius: 24px;
      box-shadow:
        0 20px 45px rgba(15, 23, 42, 0.45),
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

    .title {
      font-size: 20px;
      font-weight: 600;
      letter-spacing: 0.02em;
    }

    .tag {
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.12em;
      padding: 6px 10px;
      border-radius: 999px;
      border: 1px solid rgba(219, 234, 254, 0.35);
      background: rgba(15, 23, 42, 0.3);
    }

    .card-body {
      padding: 40px 32px 30px;
      text-align: center;
    }

    .region-label {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      padding: 6px 14px;
      border-radius: 999px;
      background: var(--primary-soft);
      color: #065f46;
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

    .region-name {
      color: var(--primary);
    }

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

    @media (max-width: 640px) {
      .card-body {
        padding: 28px 20px 24px;
      }
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
          <span>Secondary Region</span>
        </div>

        <h1>
          Served from:
          <span class="region-name">Region B (us-west-2)</span>
        </h1>

        <p class="subtitle">
          Traffic is currently landing in the <strong>Oregon</strong> AWS Region
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
<img width="975" height="439" alt="image" src="https://github.com/user-attachments/assets/2d1c22b3-7bd2-4dad-9498-0938a28f37f8" />

 - Launch the Instance.

# Step 7: Create the Target Group
 - In the EC2 console, go to Target Groups → Create target group
 - Target type: Instance
 - Name: tg-region-b
 - Protocol & Port: HTTP, port 80
 - VPC: Ensure it shows multi-region-vpc-b
<img width="975" height="499" alt="image" src="https://github.com/user-attachments/assets/6a659dfb-b795-426a-bc55-4820dcd3049f" />

 - Click Next and register your EC2 Instance.
 - From the list of instances, select web-server-b
 - Click Include as pending
<img width="975" height="421" alt="image" src="https://github.com/user-attachments/assets/89320abb-7fe6-4a23-9325-6bbbe60fe80a" />

 - Click Next then Create target group

# Step 8: Create the Application Load Balancer (ALB)
 - Open the EC2 Console → Load Balancers
 - Click Create Load Balancer
 - Select Application Load Balancer
 - Enter the name: alb-region-b
 - Scheme: Internet-facing 
<img width="975" height="396" alt="image" src="https://github.com/user-attachments/assets/a6cd399b-de7b-444e-95c8-8b2995453bc5" />

 - VPC: Select multi-region-vpc-b
 - Under Mappings, choose both public subnets:
    - subnet-b1
    - subnet-b2
 - Under Security Groups: Select web-sg-b
<img width="975" height="467" alt="image" src="https://github.com/user-attachments/assets/69a08220-577b-4460-8130-e779a8d4d49e" />

 - Target group: tg-region-b
<img width="975" height="474" alt="image" src="https://github.com/user-attachments/assets/cf0e7023-9e0b-42b9-b1c7-7142aa03429a" />

 - Create load balancer

# Step 9: Verify Everything Works
Once the ALB status becomes Active:

 - Open the EC2 Console → Load Balancers
 - Select alb-region-b
 - Copy the DNS name from the details panel
 - Paste it into your browser

Now the Region B web application is being shown:
<img width="975" height="353" alt="image" src="https://github.com/user-attachments/assets/6578681a-8f78-4529-94ba-34a1998de9a9" />





