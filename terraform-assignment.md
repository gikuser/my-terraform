# 1️⃣ What Terraform Is (Exam Explanation)

**Terraform** is an *Infrastructure as Code (IaC)* tool by HashiCorp that lets you **define, provision, and manage cloud infrastructure using declarative configuration files**.

Instead of clicking in AWS Console:

* You **write code**
* Terraform **figures out dependencies**
* Terraform **creates, updates, or destroys resources automatically**

Key concepts:

* **Provider** → Cloud platform (AWS)
* **Resource** → AWS objects (VPC, EC2, IAM, etc.)
* **State** → Tracks what Terraform created
* **Plan → Apply** workflow

---

# 2️⃣ How Terraform Projects Are Structured

A **clean, exam-perfect Terraform project** for this assignment would look like this:

```
cs423-assignment-4/
│
├── provider.tf
├── variables.tf
├── iam.tf
├── network.tf
├── security.tf
├── keypair.tf
├── ec2.tf
├── outputs.tf
├── user_data.sh
├── terraform.tfvars
├── README.md
```

**Why multiple files?**
Terraform doesn’t require it, but **best practice** is:

* Easier to read
* Easier to explain
* Easier to maintain

---

# 3️⃣ Terraform Workflow (VERY IMPORTANT FOR EXAM)

This is the **exact sequence** you’d explain if asked *“How would you deploy this?”*

1. Install Terraform
2. Configure AWS credentials
3. Write Terraform files
4. Initialize Terraform

   ```
   terraform init
   ```
5. Preview changes

   ```
   terraform plan
   ```
6. Deploy infrastructure

   ```
   terraform apply
   ```
7. Terraform stores **state**
8. Outputs are displayed (IPs, IAM user, etc.)

---

# 4️⃣ Provider Configuration (provider.tf)

This tells Terraform **which cloud** to use.

```hcl
provider "aws" {
  region = "us-east-1"
}
```

📌 Exam note:

* Terraform uses AWS SDK internally
* Credentials are taken from environment variables or AWS CLI config

---

# 5️⃣ Variables (variables.tf & terraform.tfvars)

## variables.tf

Defines inputs.

```hcl
variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

variable "instance_type" {
  default = "t2.micro"
}
```

## terraform.tfvars

Actual values (optional).

```hcl
vpc_cidr = "10.0.0.0/16"
```

📌 Exam benefit:

* Makes code reusable
* Separates configuration from logic

---

# 6️⃣ Task 1 – IAM User (iam.tf)

### What AWS Resource?

`aws_iam_user`, `aws_iam_user_login_profile`, `aws_iam_user_policy_attachment`

### Why?

* Terraform needs permissions
* Assignment requires IAM creation via Terraform

```hcl
resource "aws_iam_user" "terraform_user" {
  name = "terraform-cs423-devops"
}

resource "aws_iam_user_login_profile" "login" {
  user = aws_iam_user.terraform_user.name
  password_reset_required = true
}

resource "aws_iam_user_policy_attachment" "admin" {
  user       = aws_iam_user.terraform_user.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
}
```

📌 Exam explanation:

> Terraform creates an IAM user with console access and attaches AdministratorAccess so it can manage AWS resources.

---

# 7️⃣ Task 2 – Networking (network.tf)

This is **the most important section**.

---

## 7.1 VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "devops-assignment-4"
  }
}
```

✔ /16 allows **65,536 IPs**

---

## 7.2 Subnets (Public + Private in 2 AZs)

Each subnet supports ~255 machines → `/24`

```hcl
resource "aws_subnet" "public_1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "cs423-devops-public-1"
  }
}
```

Same for:

* public_2
* private_1
* private_2

📌 Exam explanation:

* Public subnets auto-assign public IPs
* Private subnets do NOT

---

## 7.3 Internet Gateway

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}
```

---

## 7.4 Route Tables

### Public Route Table

```hcl
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
```

### Private Route Table (NO internet route)

```hcl
resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id
}
```

📌 Exam explanation:

* Private route table has **no default route**
* Therefore no internet access

---

# 8️⃣ Task 3 – Security Groups (security.tf)

Principle of Least Privilege.

```hcl
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["YOUR_IP/32"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

📌 Exam explanation:

* SSH only from your IP
* HTTP open to public
* Egress fully allowed

---

# 9️⃣ Task 3 – Key Pair (keypair.tf)

Terraform **creates** a key pair from a local public key.

```hcl
resource "aws_key_pair" "key" {
  key_name   = "cs423-assignment4-key"
  public_key = file("~/.ssh/id_rsa.pub")
}
```

📌 Exam note:

* Private key stays on your machine
* Public key stored in AWS

---

# 🔟 Task 4 – EC2 Instances (ec2.tf)

---

## 10.1 User Data Script (user_data.sh)

```bash
#!/bin/bash
apt update -y
apt install apache2 -y
systemctl start apache2
systemctl enable apache2
echo "<h1>CS423 DevOps Assignment</h1>" > /var/www/html/index.html
```

Terraform injects this at launch.

---

## 10.2 Web Server EC2 (Public Subnet)

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public_1.id
  key_name      = aws_key_pair.key.key_name
  security_groups = [aws_security_group.web_sg.id]

  user_data = file("user_data.sh")

  tags = {
    Name = "Assignment4-EC2-1"
  }
}
```

---

## 10.3 Backend EC2 (Private Subnet)

```hcl
resource "aws_instance" "backend" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.private_1.id
  key_name      = aws_key_pair.key.key_name

  tags = {
    Name = "Assignment4-EC2-2"
  }
}
```

📌 Exam explanation:

* Public EC2 → Web server
* Private EC2 → Database / ML / backend
* Communicate internally via private IP

---

# 1️⃣1️⃣ Task 5 – Outputs (outputs.tf)

```hcl
output "web_public_ip" {
  value = aws_instance.web.public_ip
}

output "backend_private_ip" {
  value = aws_instance.backend.private_ip
}

output "iam_user" {
  value = aws_iam_user.terraform_user.name
}
```

📌 Exam benefit:

* Outputs show results without logging into AWS

---

# 1️⃣2️⃣ README.md (What You’d Explain)

Your README would describe:

* Architecture overview
* How to initialize Terraform
* How to access the web server
* SSH command example

Example:

```
ssh -i cs423-assignment4-key.pem ubuntu@<public-ip>
```

---

# 1️⃣3️⃣ Architecture Diagram (Exam Explanation)

Textually:

```
Internet
   |
Internet Gateway
   |
Public Subnet (AZ-1)
   |
EC2 Web Server (Apache :80)
   |
Private Subnet (AZ-1)
   |
EC2 Backend
```

---

# 1️⃣4️⃣ How You’d Explain This in the Exam (KEY)

If asked:

> “Explain how Terraform was used in this project”

You say:

> We used Terraform to define AWS infrastructure declaratively. The project provisions an IAM user, a VPC with public and private subnets across two availability zones, an internet gateway, route tables, security groups, key pairs, and EC2 instances. Terraform manages dependencies automatically and provides outputs for IP addresses and IAM details.

---

If you want next:

* ❓ **Viva-style questions & answers**
* 📄 **Exam-ready one-page summary**
* 🧠 **“Explain this in 2 minutes” script**

Just tell me.
