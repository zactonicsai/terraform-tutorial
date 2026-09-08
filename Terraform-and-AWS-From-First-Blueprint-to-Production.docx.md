  
**Terraform & AWS**

From First Blueprint to Production

A step-by-step book built around a real project:  
*Kafka \+ Kafka UI \+ Keycloak on AWS EC2, deployed with modular Terraform*

Written for beginners. Explained with schools, grocery stores, and construction sites.  
September 2026 edition

# **Table of Contents**

# **![][image1]How to Use This Book**

This book teaches two things at once: **Terraform** (a tool that builds cloud infrastructure from text files) and **AWS** (Amazon's cloud, where that infrastructure lives). Instead of teaching with toy examples that you throw away, the whole book is built around one real project: a small Kafka messaging system with a web dashboard (Kafka UI) protected by a login server (Keycloak), all running on AWS.

We start simple. Every chapter builds on the one before it. By the end you will have:

* Read every line of the project and understood **what** it does, **why** it is there, and **how** it works.

* Deployed the project yourself, step by step, and verified it with the AWS command line.

* Learned the best practices, common mistakes ("gotchas"), and tips that professionals use.

* Seen how to grow the project into something that a real company could run in production, and how to test it.

## **Who this book is for**

You do not need to know anything about Terraform or AWS. If you can use a computer, type commands into a terminal, and follow a recipe, you are ready. Concepts are explained the way you would explain them to a curious middle-school student, using things you already understand: schools, grocery stores, and construction sites.

## **The three analogies we use everywhere**

| REAL-WORLD ANALOGY A school. A school has a building (the network), classrooms (subnets), hall monitors who check passes at doors (security groups), teachers and students who do the actual work (servers), and a principal's office that decides who is allowed to do what (IAM). |
| :---- |

| REAL-WORLD ANALOGY A grocery store. The store lot is the private network, the aisles are subnets, the front doors are the internet gateway, the security guard at the door is a security group, and the cash registers are servers. Employee badges are IAM roles. |
| :---- |

| REAL-WORLD ANALOGY A construction site. Terraform is the architect's blueprint plus the general contractor. You draw what you want; the contractor figures out the order to build it, builds only what has changed, and keeps a log of exactly what exists on site. Modules are the pre-built pieces (windows, doors, roof trusses) that the contractor reuses across many buildings. |
| :---- |

## **Chapter layout**

Every chapter follows the same pattern:

1. **The idea** – what the concept is, with an analogy.

2. **The details** – keywords, syntax, and examples.

3. **In our project** – where the concept appears in the real code.

4. **Verify it** – AWS CLI commands to see the real thing in your account.

5. **Tips, gotchas, best practices** – highlighted boxes.

6. **Chapter quiz** – five questions. Answers are in Appendix C.

## **Conventions**

* Text in this font is code, a file name, a command, or a keyword.

* Boxes labelled TIP, GOTCHA, NOTE, BEST PRACTICE, ANALOGY and OFFICIAL DOCS call out things worth remembering.

* Commands are written for a Mac or Linux terminal. On Windows, use WSL2 or PowerShell with the same commands.

# **Chapter 1: Infrastructure as Code**

## **The idea**

Imagine you are the principal of a brand-new school. You need classrooms, desks, a front office, hall monitors, and a rule book that says who is allowed where. There are two ways to set this up.

**The manual way.** You walk around, order furniture, tell each hall monitor what to do, and write nothing down. It works — until you need a second school. Then you have to remember everything you did and do it again by hand, and you will probably forget something.

**The blueprint way.** You write down exactly what the school should look like: 12 classrooms with 30 desks each, two doors with monitors, this rule book. Now anyone can build the same school from the paper, you can compare the paper to the real building to find differences, and if the paper changes you know precisely what to fix.

**Infrastructure as Code (IaC)** is the blueprint way for computer systems. Instead of clicking around a website to create servers and networks, you write text files that describe what you want, and a tool builds it for you. Terraform is the most popular tool for doing this.

## **Why it matters**

| Manual clicking | Infrastructure as Code |
| :---- | :---- |
| Hard to repeat exactly | Same result every time |
| No record of what was done | Files live in Git, with full history |
| One person knows how it works | Anyone can read the files |
| Mistakes found in production | Mistakes found in review, before building |
| Changing one thing is scary | The tool shows exactly what will change |
| Tearing down is a treasure hunt | One command removes everything |

## **The three big ideas behind Terraform**

**1\. Declarative.** You describe the *end state* ("I want a server this big in this network"), not the steps. Terraform figures out the steps. This is like ordering a sandwich by saying "turkey on wheat with lettuce" instead of "walk to the fridge, pick up the bread…".

**2\. State.** Terraform keeps a file (the state) that remembers what it built and the real IDs AWS assigned. This is the contractor's inventory log. Without it, Terraform would not know that the server it sees in AWS is the one from your blueprint.

**3\. Plan before apply.** Terraform always shows you what it *will* do before doing it. You read the plan, and only then say yes.

| REAL-WORLD ANALOGY A construction contractor reads the new blueprint, compares it to the building as it stands, and hands you a change order: "Add 2 windows, remove 1 door, repaint the lobby." You approve the change order before a single hammer swings. That change order is a Terraform plan. |
| :---- |

## **What "the cloud" actually is**

The cloud is just other people's computers that you rent by the minute. Amazon has enormous buildings full of computers all over the world. When you ask AWS for a server, a small slice of one of those computers is handed to you within seconds. You pay only while you use it, and you can give it back any time. IaC is the natural way to talk to the cloud, because everything in the cloud is created by an API (a programmable interface), and Terraform speaks that API for you.

## **Our project in one paragraph**

We are going to build a tiny "company" in the cloud. It has a private network (a **VPC**) with one public room (a **subnet**). Inside are two servers (**EC2 instances**). One runs **Keycloak**, a login service that checks usernames and passwords. The other runs **Kafka**, a message-passing system, plus **Kafka UI**, a website for looking at Kafka. Kafka UI asks Keycloak to log people in. Security groups act as door guards, IAM gives the servers a badge so we can remotely log in without passwords, and Elastic IPs give each server a permanent public address.

## 

## **Chapter Quiz**

**Q1. What does "declarative" mean in Terraform?**

a) You write the steps in order

b) You describe the end result and Terraform figures out the steps

c) You must click in the AWS console first

d) Terraform guesses what you want

**Q2. What is the Terraform state file most like?**

a) A blueprint

b) A change order

c) The contractor's inventory log of what has been built

d) A receipt for payment

**Q3. Why is "plan before apply" important?**

a) It makes Terraform faster

b) It lets you review exactly what will change before anything happens

c) It is required by AWS

d) It saves money on storage

**Q4. Which is NOT a benefit of Infrastructure as Code?**

a) Repeatability

b) Version history in Git

c) Never needing to understand what is being built

d) Easy teardown

**Q5. In the school analogy, what are subnets?**

a) The principal's office

b) The classrooms

c) The hall monitors

d) The students

# **Chapter 2: AWS Basics — The Services We Use**

Before reading Terraform code, you need to know what each AWS piece is. This chapter explains every service in the project, how they fit together, and what depends on what.

## **Regions and Availability Zones**

AWS divides the world into **Regions** (like us-east-1 in Virginia or eu-west-1 in Ireland). Each region is a group of separate data centers called **Availability Zones (AZs)**, such as us-east-1a, us-east-1b. AZs are far enough apart that a fire or power failure in one does not affect the others.

| REAL-WORLD ANALOGY A grocery chain has stores in many cities (regions). Within a city, it has several stores across town (AZs). If one store loses power, the others still sell groceries. |
| :---- |

Our project picks one region and then chooses one AZ that offers the server sizes we asked for. Not every AZ has every size, just as not every store carries every product.

## **Accounts and IAM (Identity and Access Management)**

An **AWS account** is your company's space. **IAM** decides who and what may do things inside it.

* An **IAM user** is a person with a login.

* An **IAM role** is a badge that a server or service can wear to get permissions. Roles have no password; they are "assumed."

* A **policy** is a rule list: "may read from S3", "may talk to SSM."

* An **instance profile** is the holder that lets an EC2 server wear a role.

| REAL-WORLD ANALOGY In a school, the principal's office (IAM) decides who gets a key to the supply closet. Teachers (roles) get keys based on their job. A visitor badge (instance profile) is what lets a substitute teacher (EC2 server) use a teacher's key ring. |
| :---- |

In our project, both servers wear a role that allows **SSM Session Manager**, so we can open a terminal on them from our laptop without SSH keys or open ports.

## **VPC (Virtual Private Cloud)**

A **VPC** is your own private network inside AWS. It has an address range written in CIDR notation, such as 10.40.0.0/16, which means "all addresses starting with 10.40" — about 65,000 addresses.

| REAL-WORLD ANALOGY The VPC is the fenced grocery store property. Nothing gets in or out except through doors you build. |
| :---- |

### **CIDR notation in two minutes**

10.40.0.0/16 means: the first 16 bits (10.40) are fixed; the rest are yours to assign. /24 fixes 24 bits (10.40.1) leaving 256 addresses. Smaller number after the slash \= bigger network.

| CIDR | Addresses | Typical use |
| :---- | :---- | :---- |
| /16 | 65,536 | A whole VPC |
| /24 | 256 | One subnet |
| /32 | 1 | One specific computer (used in "allow only my IP") |

## **Subnets**

A **subnet** is a slice of the VPC's addresses that lives in exactly one AZ. A **public** subnet has a route to the internet; a **private** subnet does not.

| REAL-WORLD ANALOGY Subnets are aisles in the store. The bakery aisle by the front door is "public" — customers walk right in. The stock room is "private" — only staff, through the back. |
| :---- |

## **Internet Gateway and Route Tables**

An **Internet Gateway (IGW)** is the door between your VPC and the internet. A **route table** is the set of signs that say "to reach the internet, go through this door." A subnet becomes public when its route table has a 0.0.0.0/0 route (meaning "everywhere else") pointing at the IGW.

| REAL-WORLD ANALOGY The IGW is the front entrance. The route table is the sign at the end of each aisle: "Exit this way." Without a sign, shoppers in that aisle never find the exit. |
| :---- |

**NAT Gateway** (optional, not used by default in our project) lets private subnets reach out to the internet (to download updates) without letting the internet in. Like a mail slot: staff can send letters out, nobody can climb in.

## **Security Groups**

A **security group (SG)** is a firewall attached to a server's network card. It has **ingress** rules (who may come in, on which port) and **egress** rules (where the server may go). Security groups are *stateful*: if you allow a request in, the reply is automatically allowed out.

| REAL-WORLD ANALOGY A security group is the hall monitor at a classroom door. The rule list says "students with a hall pass from room 8080 may enter." The monitor remembers who came in, so they can leave without a second pass. |
| :---- |

A **port** is a numbered door on a server. Web servers use 80/443, Kafka UI uses 8080, Keycloak uses 8443 and 9000\. Rules in our project:

| Server | Port | Who may come in | Why |
| :---- | :---- | :---- | :---- |
| Kafka | 8080 | Your IP (allowed\_cidr) | Kafka UI website |
| Keycloak | 8443 | Your IP | Keycloak login pages and admin console (HTTPS) |
| Keycloak | 8443 | The Kafka security group | Kafka UI checks tokens with Keycloak privately |
| Keycloak | 9000 | Your IP | Health and metrics endpoints |

## **EC2 (Elastic Compute Cloud)**

**EC2** is the service that rents you servers, called **instances**. Key ideas:

* **Instance type** – size. t3.small (2 vCPU, 2 GB RAM), t3.medium (2 vCPU, 4 GB RAM).

* **AMI (Amazon Machine Image)** – the "install disk." We use Amazon Linux 2023\.

* **EBS volume** – the hard drive. We use encrypted gp3 disks.

* **User data** – a script that runs once at first boot. Ours installs Docker and starts the apps.

* **Instance metadata (IMDS)** – a tiny web service inside every instance that tells it about itself (its IP, its role). We require the safer version, IMDSv2.

| REAL-WORLD ANALOGY An instance type is the size of the truck you rent. The AMI is what is already loaded in the truck when you pick it up. User data is the note taped to the dashboard: "When you start, drive to the store and set up the display." |
| :---- |

## **Launch Templates**

A **launch template** is a saved recipe for creating an instance: which AMI, what size, which subnet, which security groups, which user data, what disk. Instead of typing all of that each time, you point at the template.

| REAL-WORLD ANALOGY A construction company keeps a standard "classroom package" spec: these windows, this flooring, these lights. Every new classroom is built from the package. Change the package, and the next classroom gets the change. |
| :---- |

## **Elastic IP (EIP)**

Instances normally get a random public IP that changes when they restart. An **Elastic IP** is a permanent public address you own and attach to an instance. We allocate EIPs *before* building the servers, because the servers' startup scripts need to know their own public address.

| REAL-WORLD ANALOGY A store's street address stays the same even if the building is remodeled. The EIP is the street address; the instance is the building. |
| :---- |

## **SSM (Systems Manager)**

**SSM** does two jobs here:

* **Parameter Store** – a public dictionary where AWS publishes the latest AMI IDs. We read /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86\_64 to always get the newest Amazon Linux.

* **Session Manager** – opens a shell on an instance through AWS, using the instance's IAM role. No SSH port needed.

## **How the pieces fit together**

Reading from the outside in:

1. The **Region** is chosen; inside it we pick an **AZ**.

2. The **VPC** is created with address range 10.40.0.0/16.

3. An **Internet Gateway** is attached to the VPC.

4. A **public subnet** 10.40.1.0/24 is created in the chosen AZ; its **route table** sends 0.0.0.0/0 to the IGW.

5. Two **security groups** are created inside the VPC.

6. An **IAM role** with the SSM policy is wrapped in an **instance profile**.

7. Two **Elastic IPs** are allocated.

8. Two **launch templates** combine AMI \+ size \+ subnet \+ SG \+ profile \+ user data.

9. Two **EC2 instances** are launched from the templates and attached to the EIPs.

10. At boot, **user data** installs Docker and starts Keycloak, then Kafka and Kafka UI.

## **Dependency map**

Terraform builds things in the right order because each resource *references* the one it needs. Here is the full list for our project — what each thing needs and why.

| Resource | Depends on | Why |
| :---- | :---- | :---- |
| VPC | (nothing) | The foundation |
| Internet Gateway | VPC | Must attach to a VPC |
| Subnet | VPC, selected AZ | Lives inside the VPC's address space in one AZ |
| Route table | VPC | Belongs to the VPC |
| Route (0.0.0.0/0) | Route table, IGW | Points the table at the door |
| Route table association | Subnet, route table | Connects the sign to the aisle |
| Security group | VPC | Firewalls are per-VPC |
| SG rule (SG-to-SG) | Both security groups | The Keycloak rule references the Kafka SG's ID |
| IAM role | (nothing) | Global, not tied to a region |
| Policy attachment | IAM role | Attaches to the role |
| Instance profile | IAM role | Wraps the role for EC2 |
| Elastic IP | (nothing) | Allocated from AWS's pool |
| AMI lookup | (nothing) | Reads a public SSM parameter |
| Launch template | AMI, subnet, SG, instance profile, user data | The recipe needs all ingredients |
| Keycloak instance | Keycloak launch template | Built from the recipe |
| Keycloak EIP association | Keycloak instance, Keycloak EIP | Needs both to exist |
| Kafka user data | Keycloak instance's private IP | Kafka UI must know where Keycloak is |
| Kafka launch template | Kafka user data \+ same as above |  |
| Kafka instance | Kafka launch template, **Keycloak instance** | Kafka UI fetches Keycloak's certificate at boot, so Keycloak must be running first |
| Kafka EIP association | Kafka instance, Kafka EIP |  |

| NOTE Most dependencies are implicit — Terraform sees vpc\_id \= aws\_vpc.this.id and knows the subnet needs the VPC. A few are explicit with depends\_on, used when there is no attribute reference but ordering still matters (Kafka waiting for Keycloak). |
| :---- |

## **Verify with the AWS CLI**

You will run these commands throughout the book. Set your region once:

export AWS\_DEFAULT\_REGION=us-east-1  
aws sts get-caller-identity          \# who am I?  
aws ec2 describe-availability-zones \--query 'AvailabilityZones\[\].ZoneName'  
aws ec2 describe-vpcs \--query 'Vpcs\[\].{Id:VpcId,Cidr:CidrBlock,Name:Tags\[?Key==\`Name\`\].Value|\[0\]}' \--output table  
aws ec2 describe-subnets \--query 'Subnets\[\].{Id:SubnetId,Cidr:CidrBlock,AZ:AvailabilityZone,Public:MapPublicIpOnLaunch}' \--output table  
aws ec2 describe-security-groups \--query 'SecurityGroups\[\].{Id:GroupId,Name:GroupName,Vpc:VpcId}' \--output table  
aws ec2 describe-instances \--query 'Reservations\[\].Instances\[\].{Id:InstanceId,Type:InstanceType,State:State.Name,IP:PublicIpAddress}' \--output table  
aws iam list-roles \--query 'Roles\[?contains(RoleName, \`kafka\`)\].RoleName'

| TIP \--query uses a language called JMESPath to pick fields. \--output table makes results readable. Add \--output json when you want to copy IDs. |
| :---- |

| OFFICIAL DOCS VPC user guide: https://docs.aws.amazon.com/vpc/latest/userguide/ · EC2 user guide: https://docs.aws.amazon.com/ec2/ · IAM: https://docs.aws.amazon.com/IAM/latest/UserGuide/ · Security groups: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html · Session Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html |
| :---- |

## **Chapter Quiz**

**Q1. A subnet lives in how many Availability Zones?**

a) Exactly one

b) Two, for safety

c) All of them

d) It depends on size

**Q2. What makes a subnet "public"?**

a) Its name contains "public"

b) Its route table sends 0.0.0.0/0 to an Internet Gateway

c) It has more than 100 addresses

d) It has a security group

**Q3. How many addresses are in 10.40.1.0/24?**

a) 24

b) 256

c) 65,536

d) 1

**Q4. Why does the project allocate Elastic IPs before creating the servers?**

a) EIPs are cheaper that way

b) The servers' startup scripts need to know their permanent public address

c) AWS requires it

d) To pick the AZ

**Q5. What does an instance profile do?**

a) Stores the server's password

b) Lets an EC2 instance wear an IAM role

c) Chooses the instance type

d) Opens a port

# **Chapter 3: How Terraform Works**

## **The workflow: write, plan, apply**

Terraform has a simple rhythm you will repeat hundreds of times:

1. **Write** – edit .tf files describing what you want.

2. **Init** – terraform init downloads the *providers* (plugins that talk to AWS) and sets up the state backend.

3. **Plan** – terraform plan compares your files \+ state to reality and prints the change order.

4. **Apply** – terraform apply executes the plan (after you type yes).

5. **Destroy** – terraform destroy removes everything it built.

| REAL-WORLD ANALOGY Write \= draw the blueprint. Init \= hire subcontractors who know AWS. Plan \= the change order. Apply \= build it. Destroy \= demolition day. |
| :---- |

## **The moving parts**

| Part | What it is | Analogy |
| :---- | :---- | :---- |
| Configuration (.tf files) | Your description of resources | The blueprint |
| Provider | Plugin that knows how to call one cloud's API (hashicorp/aws) | A subcontractor who knows one trade |
| Resource | One thing to create (a VPC, an instance) | One item on the blueprint |
| Data source | Something to *read*, not create (latest AMI ID) | Looking up the building code |
| State | Record of what exists and its IDs | Inventory log |
| Backend | Where state is stored (local file, S3) | The filing cabinet |
| Plan | Computed list of create/update/delete actions | Change order |
| Graph | Internal map of what depends on what | Build schedule |

## **How Terraform decides what to do**

For every resource, Terraform compares three things: what your **files** say, what the **state** says was built last time, and what **AWS** says exists right now (it refreshes by asking AWS). Then:

* In files, not in state → **create** (\+).

* In state, not in files → **destroy** (\-).

* In both, but different → **update in place** (\~) or **replace** (\-/+) if the change cannot be done live.

* Same everywhere → do nothing.

| GOTCHA A replace destroys and recreates a resource. Changing an EC2 instance's subnet or AMI replaces it, and the disk is lost. The plan marks these with \-/+ and says "forces replacement." Always look for that phrase before typing yes. |
| :---- |

## **The dependency graph**

Terraform reads every file, finds every reference like aws\_vpc.this.id, and builds a graph. Things with no dependencies build first, in parallel (up to 10 at once by default). This is why you never write "create the VPC, *then* the subnet" — the reference does it for you.

## **State in detail**

State is a JSON file (terraform.tfstate). It contains every resource's attributes, including some sensitive values (passwords generated by Terraform). Rules:

* Never edit it by hand.

* Never commit it to Git.

* For teams, store it remotely (S3 \+ locking) so two people cannot apply at once.

* terraform state list shows what is tracked; terraform state show \<addr\> shows one resource.

| BEST PRACTICE Treat state like the school's master key cabinet: locked, backed up, and accessed only through the tool. |
| :---- |

## **Chapter Quiz**

**Q1. What does terraform init do?**

a) Creates all resources

b) Downloads providers and configures the backend

c) Deletes state

d) Runs the tests

**Q2. A plan line starting with \-/+ means:**

a) Update in place

b) Nothing changes

c) Destroy then recreate (replace)

d) Read-only

**Q3. How does Terraform know the subnet must be built after the VPC?**

a) You list them in order

b) The subnet references aws\_vpc.this.id

c) Alphabetical order

d) It builds everything at once

**Q4. Which is TRUE about state?**

a) It is safe to commit to Git

b) It may contain secrets and must be protected

c) It can be edited by hand freely

d) It is optional

**Q5. A provider is best described as:**

a) The cloud itself

b) A plugin that translates Terraform into one API's calls

c) A module

d) A variable file

# **Chapter 4: Setting Up and Your First Terraform Project**

## **Step 1 – Install the tools**

**Terraform.** Download from https://developer.hashicorp.com/terraform/install or use a package manager:

\# macOS  
brew tap hashicorp/tap && brew install hashicorp/tap/terraform  
\# Windows (PowerShell, as admin)  
choco install terraform  
\# Verify  
terraform \-version

**AWS CLI v2.** https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

aws \--version

**Session Manager plugin** (to open shells on instances): https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html

## **Step 2 – Create AWS credentials**

1. Sign in to the AWS console. Open IAM → Users → Create user (e.g. terraform-lab).

2. Attach a policy. For learning, AdministratorAccess is simplest; in real life use least privilege (see Chapter 21).

3. Create an **access key** for the user and copy both values.

4. Configure the CLI:

aws configure  
\# AWS Access Key ID: AKIA...  
\# AWS Secret Access Key: ...  
\# Default region name: us-east-1  
\# Default output format: json  
aws sts get-caller-identity

| GOTCHA Never paste access keys into .tf files or commit them to Git. Terraform's AWS provider automatically reads the same credentials the CLI uses. |
| :---- |

| BEST PRACTICE For anything beyond a personal lab, use AWS IAM Identity Center (SSO) with aws sso login, which gives temporary credentials that expire. |
| :---- |

## **Step 3 – Your very first project**

Create a folder and one file, main.tf:

terraform {  
  required\_version \= "\>= 1.6.0"  
  required\_providers {  
    aws \= {  
      source  \= "hashicorp/aws"  
      version \= "\~\> 6.0"  
    }  
  }  
}  
   
provider "aws" {  
  region \= "us-east-1"  
}  
   
resource "aws\_vpc" "first" {  
  cidr\_block \= "10.99.0.0/16"  
  tags \= {  
    Name \= "my-first-vpc"  
  }  
}  
   
output "vpc\_id" {  
  value \= aws\_vpc.first.id  
}

Read it top to bottom:

| Block | What it says |
| :---- | :---- |
| terraform { required\_version } | Refuse to run on old Terraform |
| required\_providers | We need the AWS provider, version 6.x |
| provider "aws" | Talk to region us-east-1 |
| resource "aws\_vpc" "first" | Create a VPC; first is *our* nickname for it |
| cidr\_block | An argument the AWS provider requires |
| tags | Labels; Name is what the console displays |
| output "vpc\_id" | Print the real ID after apply |

Run it:

terraform init  
terraform plan  
terraform apply  
\# type: yes  
terraform output vpc\_id  
aws ec2 describe-vpcs \--filters Name=tag:Name,Values=my-first-vpc  
terraform destroy

You just built, verified, and demolished a network in the cloud. Everything else in this book is more of the same, with more pieces.

## **Step 4 – Understand what appeared on disk**

| File/folder | Purpose | Commit to Git? |
| :---- | :---- | :---- |
| .terraform/ | Downloaded providers | No |
| .terraform.lock.hcl | Pins exact provider versions | Yes |
| terraform.tfstate | State | No (use remote state) |
| terraform.tfstate.backup | Previous state | No |

## **Chapter Quiz**

**Q1. Which command downloads the AWS provider?**

a) terraform plan

b) terraform init

c) terraform apply

d) aws configure

**Q2. Where should AWS credentials live?**

a) Inside main.tf

b) In the AWS CLI config / environment, never in code

c) In the state file

d) In a tag

**Q3. In resource "aws\_vpc" "first", what is first?**

a) The AWS name of the VPC

b) Your local nickname used to reference it in code

c) The region

d) A required keyword

**Q4. Which file should be committed to Git?**

a) terraform.tfstate

b) .terraform/

c) .terraform.lock.hcl

d) .terraform.tfstate.backup

**Q5. What does terraform destroy do?**

a) Deletes the .tf files

b) Removes every resource in state

c) Removes only the VPC

d) Clears the AWS account

# **Chapter 5: The Terraform Language — Every Keyword Explained**

Terraform files are written in **HCL** (HashiCorp Configuration Language). This chapter is the quick tour of every keyword used in the project. Chapters 7–12 then go deep: types and formats (7), variables and tfvars (8), functions (9), resources and required settings (10), writing modules (11), and what to avoid (12).

## **Blocks, arguments and expressions**

resource "aws\_subnet" "public" {   \# block type, labels  
  vpc\_id     \= aws\_vpc.main.id     \# argument \= expression  
  cidr\_block \= "10.40.1.0/24"  
  tags \= { Name \= "public" }       \# a map value  
}

A **block** has a type, zero or more labels, and a body. **Arguments** are name \= value. **Expressions** produce values: literals, references, function calls, conditionals.

## **Top-level block types**

| Keyword | Purpose | Example |
| :---- | :---- | :---- |
| terraform | Settings: versions, providers, backend | terraform { required\_version \= "\>= 1.6" } |
| provider | Configure a plugin | provider "aws" { region \= var.aws\_region } |
| resource | Create/manage something | resource "aws\_vpc" "this" { ... } |
| data | Read something existing | data "aws\_ssm\_parameter" "ami" { name \= "..." } |
| variable | An input | variable "vpc\_cidr" { type \= string } |
| output | A result to print or pass on | output "vpc\_id" { value \= aws\_vpc.this.id } |
| locals | Named intermediate values | locals { name \= "${var.project}-vpc" } |
| module | Reuse a folder of Terraform | module "vpc" { source \= "../../modules/vpc" } |
| check | Assert something after plan | check "az" { assert { condition \= ... } } |

## **Referencing things**

| Reference | Meaning |
| :---- | :---- |
| var.name | An input variable |
| local.name | A local value |
| aws\_vpc.this.id | Attribute id of resource aws\_vpc named this |
| data.aws\_ssm\_parameter.ami.value | Attribute of a data source |
| module.vpc.vpc\_id | An output of a module |
| path.module | Folder of the current file |
| each.key, each.value | Current item inside for\_each |
| count.index | Current index inside count |

## **Types**

| Type | Example |
| :---- | :---- |
| string | "hello" |
| number | 20 |
| bool | true |
| list(string) | \["a", "b"\] |
| map(string) | { Name \= "x" } |
| set(string) | Like a list, unordered, no duplicates |
| object({...}) | Structure with named fields |
| optional(type, default) | Object field that may be omitted |

A real example from the project — the keycloak variable is an object where every field is optional:

variable "keycloak" {  
  type \= object({  
    enabled          \= optional(bool, true)  
    instance\_type    \= optional(string, "t3.small")  
    root\_volume\_size \= optional(number, 20\)  
    admin\_username   \= optional(string, "admin")  
    admin\_password   \= optional(string)      \# no default \=\> null  
  })  
  default \= {}  
}

Setting keycloak \= { instance\_type \= "t3.medium" } in tfvars keeps every other default. This is how one variable can configure a whole server.

## **Meta-arguments (work on any resource or module)**

| Meta-argument | Purpose |
| :---- | :---- |
| count \= N | Make N copies (or 0 to disable). Access with \[0\]. |
| for\_each \= map\_or\_set | One copy per item, keyed by name. Access with \["key"\]. |
| depends\_on \= \[...\] | Explicit ordering when there is no reference |
| provider \= aws.other | Use an alternate provider config |
| lifecycle { ... } | create\_before\_destroy, prevent\_destroy, ignore\_changes, precondition, postcondition |

| TIP Prefer for\_each over count for lists of things. With count, removing item 0 shifts every index and Terraform recreates everything. With for\_each, removing key "b" only removes "b". |
| :---- |

| GOTCHA count and for\_each values must be known at plan time. You cannot base them on an attribute AWS only returns after creation. |
| :---- |

## **Expressions you will see**

**String interpolation:** "${var.project\_name}-vpc"

**Conditional:** var.enabled ? 1 : 0

**For expression (build a map):**

{ for k, s in aws\_subnet.public : k \=\> s.id }

**For expression (filter a list):**

\[for v in list : v if v \!= null\]

**Splat:** aws\_instance.web\[\*\].id – all IDs from a counted resource.

**Try:** try(module.kafka\[0\].id, null) – return null instead of erroring when the module has zero copies.

**Coalesce:** coalesce(var.az, local.auto\_az) – first non-null, non-empty value.

## **Functions used in the project**

| Function | Does | Example |
| :---- | :---- | :---- |
| merge(a, b) | Combine maps; later wins | merge(var.tags, { Name \= "x" }) |
| flatten(list) | Un-nest lists | Used to turn per-SG rule lists into one list |
| toset, tolist, tostring | Convert types | tostring(module.lt.latest\_version) |
| setintersection(a, b, ...) | Items in all sets | AZs that support every instance type |
| sort(list) | Alphabetical | Deterministic AZ choice |
| length(x) | Count | length(var.public\_subnets) \> 0 |
| join(sep, list) | List to string | Error messages |
| templatefile(path, vars) | Render a template file | User-data scripts |
| base64encode(s) | Encode | User data must be base64 |
| file(path) | Read a file |  |
| jsonencode(map) | Map to JSON | IAM policies |
| anytrue(list), alltrue(list) | Any/all true | Validations |
| concat(a, b) | Join lists |  |

| OFFICIAL DOCS Full function list: https://developer.hashicorp.com/terraform/language/functions |
| :---- |

## **Variables in depth**

variable "allowed\_cidr" {  
  description \= "IPv4 CIDR allowed in. YOUR\_IP/32 recommended."  
  type        \= string  
  default     \= "0.0.0.0/0"  
  sensitive   \= false  
  validation {  
    condition     \= can(cidrhost(var.allowed\_cidr, 0))  
    error\_message \= "Must be a valid CIDR like 203.0.113.10/32."  
  }  
}

Ways to set a variable, from lowest to highest priority:

1. default in the block

2. Environment variable TF\_VAR\_allowed\_cidr=...

3. terraform.tfvars (loaded automatically)

4. \*.auto.tfvars

5. \-var-file=dev.tfvars

6. \-var allowed\_cidr=...

**tfvars files** are just name \= value lines. They are the "order form" for a module: the code is the menu, the tfvars are what you ordered.

## **Outputs, locals, sensitive values**

* output values print after apply and are readable by other stacks through remote state.

* locals are for computing a value once and naming it.

* Mark passwords sensitive \= true; Terraform hides them in plans and outputs (they still exist in state).

## **Templates: templatefile and .tftpl**

A template is a text file with ${variable} placeholders. templatefile("user\_data.sh.tftpl", { ip \= "1.2.3.4" }) fills them in. Templates can also loop: %{ for x in list }...%{ endfor }.

| GOTCHA If your shell script itself contains ${...} (bash syntax), Terraform will try to interpolate it. Escape with $${...}. Our scripts avoid this by using $VARIABLE without braces for bash variables. |
| :---- |

## **Checks, preconditions and validations**

| Mechanism | Where | When it runs |
| :---- | :---- | :---- |
| validation inside variable | Variable block | As soon as the variable is evaluated; can only reference that variable (cross-variable references need Terraform 1.9+) |
| precondition in lifecycle | Resource | Before creating/updating that resource |
| check block | Top level | After plan; produces a warning, does not stop apply |

## **Chapter Quiz**

**Q1. Which meta-argument creates one copy per key of a map?**

a) count

b) for\_each

c) depends\_on

d) lifecycle

**Q2. What does try(module.kafka\[0\].id, null) return when the module has count \= 0?**

a) An error

b) An empty string

c) null

d) 0

**Q3. Which has the highest priority when setting a variable?**

a) default

b) terraform.tfvars

c) \-var on the command line

d) TF\_VAR environment variable

**Q4. optional(string, "admin") in an object type means:**

a) The field is required

b) The field may be omitted and defaults to "admin"

c) The field is always null

d) The field is a list

**Q5. Why must count be known at plan time?**

a) For speed

b) Terraform must know how many resources to plan before creating anything

c) AWS requires it

d) It doesn't have to be

# **Chapter 6: Modules — Reusable Building Blocks**

## **The idea**

A **module** is a folder of .tf files with inputs (variables) and outputs. The root folder you run terraform apply in is itself a module (the "root module"). Any other folder you call with a module block is a **child module**.

| REAL-WORLD ANALOGY A window manufacturer does not build each window from raw glass on site. They have a "double-hung window" product with options (width, height, color). The builder orders 40 of them with different options. The module is the product; the module block is the order form; variables are the options; outputs are what the builder needs to know afterwards (how big is the rough opening). |
| :---- |

## **Anatomy of a module**

modules/elastic\_ip/  
├── main.tf        \# resources  
├── variables.tf   \# inputs  
├── outputs.tf     \# results  
└── versions.tf    \# required providers

main.tf:

resource "aws\_eip" "this" {  
  domain \= "vpc"  
  tags   \= merge(var.tags, { Name \= "${var.name}-eip" })  
}

variables.tf:

variable "name" { type \= string }  
variable "tags" { type \= map(string), default \= {} }

outputs.tf:

output "allocation\_id" { value \= aws\_eip.this.id }  
output "public\_ip"     { value \= aws\_eip.this.public\_ip }

Calling it:

module "keycloak\_eip" {  
  source \= "../../modules/elastic\_ip"  
  name   \= "${var.project\_name}-keycloak"  
  tags   \= var.tags  
}  
\# use: module.keycloak\_eip.public\_ip

## **Rules of good modules**

* **No hard-coded values.** Everything that could differ between uses is a variable.

* **Sensible defaults.** A caller should be able to pass 2–3 things and get a good result.

* **Outputs for everything a caller might need** (IDs, ARNs, names, IPs).

* **One job.** A VPC module makes networks; it does not also make servers.

* **Document inputs** with description.

* **Pin providers** in versions.tf but do **not** put provider blocks inside child modules — the root passes providers down automatically.

| GOTCHA A provider block inside a child module makes it impossible to use with count/for\_each and hard to reuse. Keep providers in the root. |
| :---- |

## **Module sources**

| Source | Example |
| :---- | :---- |
| Local path | source \= "../../modules/vpc" |
| Terraform Registry | source \= "terraform-aws-modules/vpc/aws" version \= "5.x" |
| Git | source \= "git::https://github.com/org/repo.git//modules/vpc?ref=v1.2.0" |

| BEST PRACTICE Pin a version (?ref= or version \=) for anything not local. An unpinned module can change under you. |
| :---- |

## **Modules with count and for\_each**

Modules accept count and for\_each just like resources (Terraform ≥ 0.13). Our project uses count \= var.keycloak.enabled ? 1 : 0 to make a server optional, and for\_each \= var.instance\_roles to build any number of IAM roles from a map.

## **Stacks vs modules**

A **stack** (our word for a root module with its own state) is a deployable unit. We split the project into two stacks:

* 10-network – slow-changing, foundational, shared.

* 20-apps – changes often, depends on the network.

Splitting means an app change can never accidentally destroy the network, and two teams can own the two halves. The stacks share information through **outputs** and **remote state** (Chapter 19).

## **Chapter Quiz**

**Q1. What is the root module?**

a) The modules/ folder

b) The folder where you run terraform apply

c) The AWS provider

d) The first resource

**Q2. Which should NOT be inside a reusable child module?**

a) variables.tf

b) outputs.tf

c) A provider block with a region

d) versions.tf

**Q3. How does a caller read a module's result?**

a) module.name.output\_name

b) var.name

c) resource.name

d) data.module.name

**Q4. Why split the project into a network stack and an apps stack?**

a) Terraform requires it

b) To isolate risk and let the pieces change independently

c) To use fewer files

d) To avoid modules

**Q5. What does count \= var.kafka.enabled ? 1 : 0 on a module do?**

a) Makes two copies

b) Makes the module optional based on a variable

c) Errors

d) Sets the instance type

# **Chapter 7: Types and Values — Strings, Numbers, Lists, Maps and Objects**

Everything in Terraform is a value with a type. Getting types right is half of writing Terraform. This chapter shows every type, how to write it, how to read it back, and how the project uses it.

## **The primitive types**

| Type | How you write it | Notes |
| :---- | :---- | :---- |
| string | "hello" | Always double quotes. Single quotes are an error. |
| number | 20, 3.5, \-1 | No quotes. "20" is a string, but Terraform converts automatically where a number is expected. |
| bool | true, false | No quotes. |
| null | null | "No value." An argument set to null is the same as not setting it. |

variable "root\_volume\_size" { type \= number, default \= 20 }  
variable "enabled"          { type \= bool,   default \= true }  
variable "key\_name"         { type \= string, default \= null }   \# optional: null means "don't set"

| TIP null is how you make a resource argument optional. In the launch template module, key\_name \= var.key\_name with a null default means "no key pair" — AWS never sees the argument. |
| :---- |

## **Strings in depth**

### **Interpolation**

Put an expression inside ${ } to build a string:

"${var.project\_name}-vpc"                 \# kafka-keycloak-lab-vpc  
"https://${local.keycloak\_public\_ip}:8443/admin/"  
"aws ssm start-session \--region ${var.aws\_region} \--target ${module.keycloak\[0\].id}"

| GOTCHA Do not wrap a bare reference in quotes and braces: "${var.name}" is the same as var.name and terraform fmt will complain. Only use ${} when you are joining text and values. |
| :---- |

### **Escaping**

| You want | Write |
| :---- | :---- |
| A literal ${ | $${ |
| A literal %{ | %%{ |
| A double quote inside a string | \\" |
| A newline | \\n (or use a heredoc) |

Our user-data templates contain bash variables like $STACK\_DIR (no braces), so they never collide with Terraform's ${}.

### **Heredocs (multi-line strings)**

description \= \<\<-EOT  
  Map of security groups keyed by short name. Each rule must set exactly one  
  source: cidr\_ipv4, cidr\_ipv6, source\_security\_group\_key ...  
EOT

\<\<-EOT (with the dash) strips the leading indentation. Use heredocs for descriptions, inline policies and small scripts; use templatefile() for anything longer.

### **The format function — printf-style strings**

format("%s-%s", var.project\_name, "vpc")        \# "kafka-keycloak-lab-vpc"  
format("%03d", 7\)                                \# "007"  
format("%.2f GB", 1.5)                           \# "1.50 GB"  
format("%-10s|", "abc")                          \# "abc       |"  
formatlist("server-%02d", \[1, 2, 3\])             \# \["server-01", "server-02", "server-03"\]

| Verb | Meaning |
| :---- | :---- |
| %s | string |
| %d | integer |
| %f / %.2f | float / with 2 decimals |
| %q | quoted string |
| %v | any value |
| %t | boolean |
| %03d | zero-padded to width 3 |
| %% | a literal percent sign |

### **String directives — logic inside strings**

"%{ if var.enabled }enabled%{ else }disabled%{ endif }"  
"%{ for ip in var.ips }${ip},%{ endfor }"

Directives are most useful inside .tftpl template files. Add \~ to strip whitespace: %{\~ for x in list \~}.

### **Common string functions**

| Function | Example | Result |
| :---- | :---- | :---- |
| upper, lower, title | upper("kafka") | "KAFKA" |
| trimspace | trimspace("  a  ") | "a" |
| trimprefix, trimsuffix | trimsuffix("x-sg", "-sg") | "x" |
| replace | replace("a.b.c", ".", "-") | "a-b-c" |
| split | split(",", "a,b") | \["a", "b"\] |
| join | join(", ", \["a", "b"\]) | "a, b" |
| substr | substr("abcdef", 0, 3\) | "abc" |
| length | length("abc") | 3 |
| regex | regex("\[0-9\]+", "sg-123") | "123" |
| startswith, endswith | startswith("sg-1", "sg-") | true |

## **Lists and tuples**

A **list** is an ordered sequence of values of the same type. Index from 0\.

variable "instance\_types\_for\_az\_selection" {  
  type    \= list(string)  
  default \= \["t3.medium", "t3.small"\]  
}  
var.instance\_types\_for\_az\_selection\[0\]       \# "t3.medium"  
length(var.instance\_types\_for\_az\_selection)  \# 2

A **tuple** is like a list but each position may have a different type: \["a", 1, true\]. You rarely declare tuples; they appear when Terraform infers a type from a literal.

| Function | Example | Result |
| :---- | :---- | :---- |
| concat | concat(\["a"\], \["b"\]) | \["a", "b"\] |
| contains | contains(\["a","b"\], "a") | true |
| distinct | distinct(\["a","a","b"\]) | \["a","b"\] |
| element | element(\["a","b"\], 3\) | "b" (wraps around) |
| index | index(\["a","b"\], "b") | 1 |
| flatten | flatten(\[\["a"\],\["b","c"\]\]) | \["a","b","c"\] |
| reverse | reverse(\[1,2\]) | \[2,1\] |
| slice | slice(\[1,2,3,4\], 1, 3\) | \[2,3\] |
| sort | sort(\["b","a"\]) | \["a","b"\] |
| compact | compact(\["a","","b"\]) | \["a","b"\] (drops empty strings) |
| chunklist | chunklist(\[1,2,3,4\], 2\) | \[\[1,2\],\[3,4\]\] |
| range | range(3) | \[0,1,2\] |

## **Sets**

A **set** is an unordered collection with no duplicates. for\_each accepts sets of strings.

for\_each \= toset(var.managed\_policy\_arns)   \# from the IAM module

| Function | Example | Result |
| :---- | :---- | :---- |
| toset | toset(\["a","a"\]) | {"a"} |
| setintersection | setintersection(\["a","b"\], \["b","c"\]) | {"b"} |
| setunion | setunion(\["a"\], \["b"\]) | {"a","b"} |
| setsubtract | setsubtract(\["a","b"\], \["a"\]) | {"b"} |
| setproduct | setproduct(\["a"\], \[1,2\]) | \[\["a",1\],\["a",2\]\] |

| GOTCHA You cannot index a set (var.myset\[0\] is an error). Convert with tolist(...) first, and sort() if you need a stable order — exactly what the network stack does with sort(tolist(setintersection(...))). |
| :---- |

## **Maps**

A **map** is a collection of key \= value pairs where every value has the same type. Keys are always strings.

tags \= {  
  Environment \= "dev"  
  Owner       \= "platform-team"  
}  
var.tags\["Environment"\]     \# "dev"  
lookup(var.tags, "Owner", "unknown")

| Function | Example | Result |
| :---- | :---- | :---- |
| keys | keys({a=1,b=2}) | \["a","b"\] |
| values | values({a=1,b=2}) | \[1,2\] |
| lookup | lookup(m, "k", "default") | value or default |
| merge | merge({a=1}, {a=2,b=3}) | {a=2,b=3} (right wins) |
| zipmap | zipmap(\["a","b"\], \[1,2\]) | {a=1,b=2} |
| tomap | tomap({a="x"}) | map(string) |

Maps are the workhorse of the project: public\_subnets, security\_groups, instance\_roles are all maps, so each item has a **name** that becomes its for\_each key and its Terraform address.

## **Objects**

An **object** is a structure with named fields of possibly different types. Objects describe "one thing with many settings."

variable "keycloak" {  
  type \= object({  
    enabled          \= optional(bool, true)  
    instance\_type    \= optional(string, "t3.small")  
    root\_volume\_size \= optional(number, 20\)  
    key\_name         \= optional(string)          \# default null  
    tags             \= optional(map(string), {})  
  })  
  default \= {}  
}  
var.keycloak.instance\_type

**Map of objects** is the most common shape for "many things with settings":

type \= map(object({  
  cidr\_block        \= string                  \# required  
  availability\_zone \= optional(string)        \# optional  
}))

| TIP optional(type) without a default gives null. optional(type, default) fills in the default when the caller omits the field. Required fields have no optional() — omitting them is an error with a helpful message. |
| :---- |

## **Type conversion**

Terraform converts primitives automatically where safe ("20" → 20, true → "true"). For collections use explicit functions: tolist, toset, tomap, tostring, tonumber, tobool. can(tonumber(x)) tests whether a conversion would work.

## **any and inferred types**

type \= any accepts anything but gives up validation. Prefer precise types; use any only for pass-through values you never inspect.

## **Reading a value's type**

Use terraform console:

\> type(var.public\_subnets)  
map(object({ cidr\_block \= string, availability\_zone \= optional(string) ... }))  
\> var.public\_subnets\["public-a"\].cidr\_block  
"10.40.1.0/24"

## **Chapter Quiz**

**Q1. Which is a valid Terraform string?**

a) 'hello'

b) "hello"

c) hello

d) hello

**Q2. format("%03d", 5\) returns:**

a) "5"

b) "005"

c) "5.000"

d) an error

**Q3. Which collection type can you NOT index with \[0\]?**

a) list

b) tuple

c) set

d) string

**Q4. In merge({a=1}, {a=2}), what is a?**

a) 1

b) 2

c) \[1,2\]

d) error

**Q5. optional(string) with no default gives what when omitted?**

a) ""

b) "optional"

c) null

d) error

# **Chapter 8: Variables, tfvars, Locals and Outputs in Depth**

## **Declaring a variable — every argument**

variable "allowed\_cidr" {  
  description \= "IPv4 CIDR allowed to reach Kafka UI and Keycloak. YOUR\_IP/32 recommended."  
  type        \= string  
  default     \= "0.0.0.0/0"  
  nullable    \= false  
  sensitive   \= false  
  ephemeral   \= false  
   
  validation {  
    condition     \= can(cidrhost(var.allowed\_cidr, 0))  
    error\_message \= "allowed\_cidr must be a valid IPv4 CIDR such as 203.0.113.10/32."  
  }  
}

| Argument | Required? | Meaning |
| :---- | :---- | :---- |
| description | No (but always write one) | Shown in docs and error messages |
| type | No (defaults to any) | Constraint; always set it |
| default | No | If absent, the caller **must** provide a value |
| nullable | No (default true) | false rejects null |
| sensitive | No | Hide in plan/output |
| ephemeral | No (1.10+) | Never stored in state/plan — for short-lived secrets |
| validation | No, repeatable | Custom rule; condition must be true |

| BEST PRACTICE A variable with no default is a required input. Use this for things a caller must decide (vpc\_id, cidr\_block). Give defaults to everything else. |
| :---- |

## **Validation examples**

validation {  
  condition     \= contains(\["gp2", "gp3", "io1", "io2"\], var.root\_volume\_type)  
  error\_message \= "root\_volume\_type must be gp2, gp3, io1 or io2."  
}  
validation {  
  condition     \= var.root\_volume\_size \>= 8 && var.root\_volume\_size \<= 16384  
  error\_message \= "root\_volume\_size must be between 8 and 16384 GiB."  
}  
validation {  
  condition     \= can(regex("^t3\\\\.|^m\[5-7\]\\\\.", var.instance\_type))  
  error\_message \= "Only t3 and m5–m7 families are approved."  
}  
validation {  
  condition     \= alltrue(\[for k, s in var.public\_subnets : can(cidrhost(s.cidr\_block, 0))\])  
  error\_message \= "Every public subnet needs a valid cidr\_block."  
}

can(expr) returns true if expr evaluates without error — the standard way to validate formats.

## **How values reach a variable**

Terraform merges sources in this order; later sources **override** earlier ones:

1. default in the variable block

2. Environment variables TF\_VAR\_\<name\> (e.g. export TF\_VAR\_aws\_region=us-west-2)

3. terraform.tfvars in the working directory (auto-loaded)

4. terraform.tfvars.json (auto-loaded)

5. \*.auto.tfvars / \*.auto.tfvars.json, alphabetical (auto-loaded)

6. \-var-file=file.tfvars on the command line, in the order given

7. \-var name=value on the command line, in the order given

| GOTCHA dev.tfvars is not auto-loaded. If you forget \-var-file=dev.tfvars, Terraform silently uses the defaults. For required variables with no default, it will prompt interactively — and CI will hang. Always pass the file. |
| :---- |

## **tfvars file format**

A .tfvars file is plain HCL with only assignments. No blocks, no variable keyword, no expressions that reference resources (functions are allowed).

\# strings, numbers, bools, null  
aws\_region   \= "us-east-1"  
volume\_size  \= 30  
enabled      \= true  
key\_name     \= null  
   
\# list  
instance\_types\_for\_az\_selection \= \["t3.medium", "t3.small"\]  
   
\# map  
tags \= {  
  Environment \= "dev"  
  Owner       \= "platform"  
}  
   
\# object (fields you omit take their optional() defaults)  
keycloak \= {  
  instance\_type    \= "t3.small"  
  root\_volume\_size \= 20  
}  
   
\# map of objects with nested lists of objects (from stacks/10-network/dev.tfvars)  
security\_groups \= {  
  kafka \= {  
    description \= "Kafka and Kafka UI EC2 access"  
    ingress \= \[  
      { description \= "Kafka UI", from\_port \= 8080, to\_port \= 8080, cidr\_ipv4 \= "ALLOWED\_CIDR" }  
    \]  
  }  
}

Rules of thumb:

* Commas between items on one line; newlines are enough when each item is on its own line.

* Trailing commas are allowed.

* Comments: \# or // for a line, /\* \*/ for blocks.

* Keys that are not valid identifiers need quotes: "my-key" \= 1.

### **JSON tfvars**

Machines (CI, other tools) often prefer JSON. dev.tfvars.json:

{  
  "aws\_region": "us-east-1",  
  "tags": { "Environment": "dev" },  
  "keycloak": { "instance\_type": "t3.small", "root\_volume\_size": 20 }  
}

### **Setting complex values on the command line**

terraform apply \-var 'tags={Environment="dev"}' \-var 'instance\_types\_for\_az\_selection=\["t3.small"\]'  
export TF\_VAR\_tags='{Environment="dev"}'

## **Layering tfvars per environment**

terraform apply \-var-file=common.tfvars \-var-file=prod.tfvars

common.tfvars holds shared values; prod.tfvars overrides only what differs. Later files win.

## **Locals — computed values**

locals {  
  name\_prefix \= "${var.project\_name}-${var.environment}"  
  common\_tags \= merge(var.tags, { Project \= var.project\_name })  
  subnet\_id   \= var.network.subnet\_id \!= null ? var.network.subnet\_id : try(local.net.public\_subnet\_ids\[var.network.subnet\_key\], null)  
}

* Locals can reference variables, resources, data sources, other locals and modules.

* They are evaluated lazily — a local that is never used costs nothing.

* Use them to give a name to any expression you write more than once, or any expression that is hard to read inline.

| BEST PRACTICE When a local grows past three lines, break it into smaller locals with descriptive names. local.security\_groups in the network stack is the upper limit of what should live in one expression. |
| :---- |

## **Outputs — every argument**

output "keycloak\_admin\_password" {  
  description \= "Generated Keycloak administrator password."  
  value       \= local.keycloak\_admin\_password  
  sensitive   \= true  
  depends\_on  \= \[module.keycloak\]  
  precondition {  
    condition     \= local.keycloak\_admin\_password \!= null  
    error\_message \= "No admin password was generated or supplied."  
  }  
}

Reading outputs:

terraform output                              \# all, sensitive hidden  
terraform output keycloak\_admin\_password      \# prints (sensitive) — refuses  
terraform output \-raw keycloak\_admin\_password \# prints the value for scripts  
terraform output \-json                        \# machine-readable, includes sensitive  
terraform output \-json security\_group\_ids | jq \-r .keycloak

Outputs are how modules and stacks communicate. In a child module, outputs become module.name.output. In a root module, outputs become readable via terraform\_remote\_state.

| GOTCHA A sensitive value that flows into a non-sensitive output causes an error: "Output refers to sensitive values." Either mark the output sensitive \= true or (rarely, deliberately) wrap with nonsensitive(). |
| :---- |

## **Sensitive values end to end**

| Where | Behaviour |
| :---- | :---- |
| variable { sensitive \= true } | Value hidden in plans; anything derived is also sensitive |
| random\_password.x.result | Sensitive automatically |
| Passed into templatefile() | Result is sensitive (our user data) |
| Launch template user\_data | Shown as (sensitive value) in plan |
| State file | Stored in **plain text** — protect the state |
| output { sensitive \= true } | Hidden by terraform output, visible with \-raw/\-json |

## **Chapter Quiz**

**Q1. A variable with no default is:**

a) Optional

b) Required — the caller must supply it

c) Always null

d) Invalid

**Q2. Which is auto-loaded?**

a) prod.tfvars

b) prod.auto.tfvars

c) vars.hcl

d) settings.tf

**Q3. When two \-var-file flags set the same variable, which wins?**

a) The first

b) The last

c) Error

d) The default

**Q4. What does can(cidrhost(var.x, 0)) do in a validation?**

a) Creates a host

b) Returns true only if var.x is a valid CIDR

c) Reserves an IP

d) Nothing

**Q5. Where are sensitive values stored unencrypted?**

a) In the plan output

b) In terraform output

c) In the state file

d) Nowhere

# **Chapter 9: Functions and Expressions Cookbook**

Terraform has no user-defined functions, but it has about 150 built-in ones. You cannot define your own; instead you combine built-ins with locals and modules. This chapter is a cookbook: each recipe shows the expression, what it produces, and where the project uses it.

## **Operators**

| Kind | Operators | Example |
| :---- | :---- | :---- |
| Arithmetic | \+ \- \* / % | var.size \* 2 |
| Comparison | \== \!= \< \> \<= \>= | length(x) \> 0 |
| Logical | \`&& |  |
| Conditional | cond ? a : b | var.enabled ? 1 : 0 |

| GOTCHA Both branches of ? : must be the same type. var.x ? "a" : 1 is an error; var.x ? "a" : null is fine. |
| :---- |

## **Recipe: make something optional**

count \= var.create\_internet\_gateway ? 1 : 0  
\# then reference with \[0\] and protect with try():  
gateway\_id \= aws\_internet\_gateway.this\[0\].id  
output "igw" { value \= try(aws\_internet\_gateway.this\[0\].id, null) }

## **Recipe: user override, else discover**

ami\_id \= var.ami\_id \!= null ? var.ami\_id : data.aws\_ssm\_parameter.ami\[0\].value

## **Recipe: build a map from a resource**

output "public\_subnet\_ids" {  
  value \= { for k, s in aws\_subnet.public : k \=\> s.id }  
}

## **Recipe: filter a list**

\[for v in var.list : v if v \!= null\]           \# drop nulls  
\[for s in var.subnets : s.id if s.public\]      \# only public

## **Recipe: transform a map of objects**

{ for k, sg in var.security\_groups : k \=\> merge(sg, { ingress \= \[for r in sg.ingress : merge(r, { cidr\_ipv4 \= r.cidr\_ipv4 \== "ALLOWED\_CIDR" ? var.allowed\_cidr : r.cidr\_ipv4 })\] }) }

Read it inside-out: for each rule, replace the placeholder; for each group, replace its rule list; produce a new map with the same keys.

## **Recipe: nested loops → flat list → for\_each**

locals {  
  rules \= flatten(\[  
    for sg\_key, sg in var.security\_groups : \[  
      for idx, r in sg.ingress : merge(r, { id \= "${sg\_key}-${idx}", sg\_key \= sg\_key })  
    \]  
  \])  
}  
resource "aws\_vpc\_security\_group\_ingress\_rule" "this" {  
  for\_each \= { for r in local.rules : r.id \=\> r }  
  ...  
}

This "flatten then key by id" pattern is the standard way to create resources from nested data.

## **Recipe: group by**

{ for s in var.subnets : s.az \=\> s.id... }     \# the ... groups values into lists per key

## **Recipe: pick the first AZ common to all instance types**

sort(tolist(setintersection(local.az\_sets...)))\[0\]

## **Recipe: conditional nested block**

dynamic "iam\_instance\_profile" {  
  for\_each \= var.iam\_instance\_profile\_name \== null ? \[\] : \[1\]  
  content { name \= var.iam\_instance\_profile\_name }  
}

Empty list \= no block; one-item list \= one block. For a list of blocks, for\_each \= var.additional\_volumes and use block\_device\_mappings.value.field inside content.

## **Recipe: splat and try**

aws\_instance.web\[\*\].id           \# list of all IDs (count resources)  
values(aws\_subnet.public)\[\*\].id  \# same idea for for\_each resources  
try(module.kafka\[0\].id, null)    \# safe lookup

## **Recipe: coalesce and defaults**

coalesce(var.az, local.auto\_az)                \# first non-null/non-empty  
coalescelist(var.sgs, \[local.default\_sg\])       \# first non-empty list  
lookup(var.tags, "Owner", "unassigned")

## **Recipe: templates**

templatefile("${path.module}/templates/user\_data.sh.tftpl", {  
  docker\_compose\_b64 \= base64encode(local.compose)  
  keycloak\_private\_ip \= local.keycloak\_private\_ip  
})

Inside the template, only the names you pass are visible. Loops:

%{ for h in hosts \~}  
server ${h.name} ${h.ip};  
%{ endfor \~}

## **Recipe: encoding**

| Function | Use |
| :---- | :---- |
| base64encode / base64decode | User data, secrets in transit |
| jsonencode / jsondecode | IAM policies, API payloads |
| yamlencode / yamldecode | Kubernetes, compose files |
| urlencode | Query strings |
| sha256, md5, filesha256 | Trigger replacement when a file changes |

policy \= jsonencode({  
  Version   \= "2012-10-17"  
  Statement \= \[{ Effect \= "Allow", Action \= "sts:AssumeRole", Principal \= { Service \= "ec2.amazonaws.com" } }\]  
})

## **Recipe: network math**

| Function | Example | Result |
| :---- | :---- | :---- |
| cidrsubnet(prefix, newbits, num) | cidrsubnet("10.40.0.0/16", 8, 1\) | "10.40.1.0/24" |
| cidrhost(prefix, n) | cidrhost("10.40.1.0/24", 10\) | "10.40.1.10" |
| cidrnetmask | cidrnetmask("10.40.0.0/16") | "255.255.0.0" |
| cidrsubnets(prefix, bits...) | cidrsubnets("10.40.0.0/16", 8, 8\) | \["10.40.0.0/24","10.40.1.0/24"\] |

Use cidrsubnet to derive subnet ranges from the VPC range instead of typing them, so they can never overlap:

public\_subnets \= { for i, az in var.azs : "public-${az}" \=\> { cidr\_block \= cidrsubnet(var.vpc\_cidr, 8, i), availability\_zone \= az } }

## **Recipe: date, numbers, misc**

| Function | Example |
| :---- | :---- |
| timestamp() | Current UTC time — avoid in resources (changes every plan) |
| formatdate("YYYY-MM-DD", timestamp()) | Formatted date |
| max, min, abs, ceil, floor | Numbers |
| uuid() | Random each plan — avoid |
| fileexists, file, fileset | Files relative to path.module |
| sensitive(x), nonsensitive(x) | Mark / unmark |
| type(x) | In console, show type |

## **Path and workspace references**

| Reference | Value |
| :---- | :---- |
| path.module | Folder of the current module (use for templatefile, file) |
| path.root | Folder of the root module |
| path.cwd | Where you ran terraform (avoid) |
| terraform.workspace | Current workspace name (default) |

## **Testing expressions**

terraform console lets you try any expression against the current state and variables:

$ terraform \-chdir=stacks/10-network console \-var-file=dev.tfvars  
\> cidrsubnet(var.vpc\_cidr, 8, 2\)  
"10.40.2.0/24"  
\> keys(var.security\_groups)  
\["kafka", "keycloak"\]  
\> local.security\_groups.keycloak.ingress\[1\].source\_security\_group\_key  
"kafka"

## **Chapter Quiz**

**Q1. Can you define your own function in Terraform?**

a) Yes, with function blocks

b) No — combine built-ins with locals and modules

c) Only in modules

d) Only in Go

**Q2. cidrsubnet("10.0.0.0/16", 8, 3\) returns:**

a) 10.0.3.0/24

b) 10.3.0.0/24

c) 10.0.0.3/32

d) 10.0.8.0/24

**Q3. What does an empty list in a dynamic block's for\_each produce?**

a) An error

b) One empty block

c) No block at all

d) A null block

**Q4. Why avoid timestamp() in a resource argument?**

a) It is slow

b) It changes every plan, causing perpetual diffs

c) It is deprecated

d) It needs a provider

**Q5. Which tool lets you evaluate expressions interactively?**

a) terraform test

b) terraform console

c) terraform show

d) terraform graph

# **Chapter 10: Resources in Depth — Required Settings and How to Read the Docs**

## **Anatomy of a resource block**

resource "aws\_subnet" "public" {          \# 1\. type   2\. local name  
  for\_each \= var.public\_subnets           \# meta-argument (optional)  
   
  vpc\_id     \= aws\_vpc.this.id            \# required argument  
  cidr\_block \= each.value.cidr\_block      \# required argument  
  availability\_zone \= "us-east-1a"        \# optional argument  
   
  tags \= { Name \= "public" }              \# optional argument (map)  
   
  lifecycle {                             \# meta-argument block (optional)  
    precondition { ... }  
  }  
   
  depends\_on \= \[aws\_internet\_gateway.this\] \# meta-argument (optional)  
}

| Part | Rule |
| :---- | :---- |
| Type (aws\_subnet) | Provider prefix \+ resource name. Must exist in the provider. |
| Local name (public) | Letters, digits, underscores, dashes. Unique per type within a module. |
| Arguments | What **you set**. Some required, most optional. |
| Attributes | What **AWS returns** (id, arn, private\_ip). Read-only; available after apply. |
| Meta-arguments | count, for\_each, depends\_on, provider, lifecycle. Work on every resource type. |

| NOTE The difference between an argument and an attribute confuses everyone at first. cidr\_block is an argument you set. id is an attribute AWS assigns. Some names are both: you can set availability\_zone and also read it back. |
| :---- |

## **Reading the provider documentation**

Every resource page at https://registry.terraform.io/providers/hashicorp/aws/latest/docs has the same layout:

1. **Example Usage** – copy this to start.

2. **Argument Reference** – each argument marked *(Required)* or *(Optional)*, with defaults. This is the list of "required settings."

3. **Attribute Reference** – what you can read after apply (id, arn, computed values).

4. **Import** – how to bring an existing resource under management.

5. **Timeouts** – if the resource supports custom timeouts.

| TIP Before writing a resource, read only the Argument Reference and note the *(Required)* items. That is the minimum block. Add optional arguments one at a time as you need them. |
| :---- |

## **Required arguments for every resource in the project**

| Resource | Required | Commonly used optional |
| :---- | :---- | :---- |
| aws\_vpc | cidr\_block (or IPAM) | enable\_dns\_support, enable\_dns\_hostnames, tags |
| aws\_internet\_gateway | (none) | vpc\_id (practically always), tags |
| aws\_subnet | vpc\_id, cidr\_block | availability\_zone, map\_public\_ip\_on\_launch, tags |
| aws\_route\_table | vpc\_id | tags (avoid inline route) |
| aws\_route | route\_table\_id, one destination (destination\_cidr\_block), one target (gateway\_id, nat\_gateway\_id, …) |  |
| aws\_route\_table\_association | route\_table\_id, and subnet\_id or gateway\_id |  |
| aws\_eip | (none) | domain \= "vpc", tags |
| aws\_nat\_gateway | subnet\_id; allocation\_id for public NAT | connectivity\_type, tags |
| aws\_security\_group | (none) | vpc\_id, name/name\_prefix, description, tags |
| aws\_vpc\_security\_group\_ingress\_rule | security\_group\_id, ip\_protocol, and one source (cidr\_ipv4, cidr\_ipv6, referenced\_security\_group\_id, prefix\_list\_id) | from\_port, to\_port (required unless protocol \-1), description |
| aws\_iam\_role | assume\_role\_policy | name/name\_prefix, description, tags |
| aws\_iam\_role\_policy\_attachment | role, policy\_arn |  |
| aws\_iam\_role\_policy | role, policy | name |
| aws\_iam\_instance\_profile | (none, but role practically) | name/name\_prefix |
| aws\_launch\_template | (none) | image\_id, instance\_type, user\_data, network\_interfaces, block\_device\_mappings, metadata\_options |
| aws\_instance | ami \+ instance\_type, **or** launch\_template | subnet\_id, vpc\_security\_group\_ids, iam\_instance\_profile, user\_data, tags |
| aws\_eip\_association | allocation\_id (or public\_ip), and instance\_id or network\_interface\_id |  |
| random\_password | length | special, override\_special, min\_upper … |

| GOTCHA "Required by Terraform" and "required by AWS" differ. aws\_security\_group has no required arguments in Terraform, but without vpc\_id it lands in the default VPC — almost never what you want. Read the description, not just the (Required) tag. |
| :---- |

## **Nested blocks vs map arguments**

Some settings are **nested blocks** (no \=), others are **maps** (with \=). The docs tell you which.

network\_interfaces {              \# nested block: no equals sign  
  subnet\_id \= var.subnet\_id  
}  
tags \= {                          \# map argument: equals sign  
  Name \= "x"  
}

A nested block that may repeat (block\_device\_mappings) can appear several times or be generated with dynamic.

## **Data sources — reading instead of creating**

data "aws\_ssm\_parameter" "ami" {  
  name \= "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86\_64"  
}  
data "aws\_ec2\_instance\_type\_offerings" "by\_type" {  
  location\_type \= "availability-zone"  
  filter { name \= "instance-type", values \= \["t3.medium"\] }  
}  
data "aws\_caller\_identity" "current" {}     \# account ID  
data "aws\_region" "current" {}  
data "aws\_availability\_zones" "available" { state \= "available" }

Data sources are refreshed on every plan. Reference with data.\<type\>.\<name\>.\<attribute\>.

## **The lifecycle block — every option**

lifecycle {  
  create\_before\_destroy \= true                 \# build replacement first (SGs, launch templates)  
  prevent\_destroy       \= true                 \# plan fails if this would be destroyed (databases, state buckets)  
  ignore\_changes        \= \[tags\["LastScanned"\], user\_data\]   \# ignore drift on these arguments  
  replace\_triggered\_by  \= \[aws\_launch\_template.this.latest\_version\]   \# replace when another thing changes  
  precondition  { condition \= ..., error\_message \= "..." }   \# checked before apply  
  postcondition { condition \= self.state \== "available", error\_message \= "..." }  \# checked after  
}

| GOTCHA ignore\_changes cannot use variables; it must be a static list. Our EC2 module had exactly this bug in an early draft and it was removed. |
| :---- |

## **moved and removed blocks — refactoring safely**

When you rename a resource or move it into a module, Terraform would plan a destroy \+ create. A moved block tells it "same thing, new address":

moved {  
  from \= aws\_vpc.main  
  to   \= module.vpc.aws\_vpc.this  
}

removed (1.7+) drops a resource from state without destroying it:

removed {  
  from \= aws\_instance.old  
  lifecycle { destroy \= false }  
}

These are how you would migrate the original single-file project into the modular one without rebuilding.

## **import block — adopting existing resources**

import {  
  to \= module.vpc.aws\_vpc.this  
  id \= "vpc-0123456789abcdef0"  
}

Run terraform plan \-generate-config-out=generated.tf to have Terraform write the resource block for you.

## **Provisioners — know them, avoid them**

provisioner "remote-exec" and local-exec run scripts during create/destroy. They are a last resort: they are not idempotent, cannot be planned, and break the declarative model. Use **user data**, SSM Run Command, or configuration tools instead. Our project uses user data only.

## **Timeouts**

timeouts {  
  create \= "10m"  
  delete \= "20m"  
}

Only for resources that support it (RDS, EKS, NAT gateways). Rarely needed for the resources here.

## **Resource addresses**

You will see and type these in state, \-target, \-replace, moved:

| Address | Meaning |
| :---- | :---- |
| aws\_vpc.this | Plain resource |
| aws\_subnet.public\["public-a"\] | for\_each instance |
| aws\_internet\_gateway.this\[0\] | count instance |
| module.vpc.aws\_vpc.this | Inside a module |
| module.instance\_roles\["ec2\_ssm"\].aws\_iam\_role.this | Inside a for\_each module |
| module.kafka\[0\].aws\_instance.this | Inside a count module |
| data.aws\_ssm\_parameter.ami\[0\] | Data source with count |

## **Chapter Quiz**

**Q1. Where do you find a resource's required arguments?**

a) Attribute Reference

b) Argument Reference, marked (Required)

c) Import section

d) The AWS console

**Q2. id on aws\_vpc is:**

a) An argument you set

b) An attribute AWS assigns

c) A meta-argument

d) A variable

**Q3. What does a moved block do?**

a) Moves the resource to another region

b) Tells Terraform a resource has a new address, avoiding destroy/create

c) Deletes it

d) Renames the AWS Name tag

**Q4. Which is a nested block, not a map?**

a) tags

b) network\_interfaces

c) description

d) instance\_type

**Q5. Why avoid provisioners?**

a) They cost money

b) They are not idempotent or plannable and break the declarative model

c) They are removed in Terraform 1.6

d) They only work on Windows

# **Chapter 11: Modules in Depth — Writing One From Scratch**

## **Step 1 – Decide the module's job**

One job, clearly stated. Good: "Create an EC2 launch template with hardened defaults." Bad: "Create the whole Kafka environment." A module that does one thing composes; a module that does everything is a copy of the root.

| REAL-WORLD ANALOGY A window factory makes windows. It does not also pour foundations. The builder (root module) orders windows, doors and trusses from different factories and assembles them. |
| :---- |

## **Step 2 – List inputs, then decide which have defaults**

Write the variables file first. Ask for each input: *Would two callers ever want different values?* If yes, it is a variable. Then: *Is there a value that is right 80% of the time?* If yes, that is the default.

variable "name"              { type \= string }                  \# required  
variable "image\_id"          { type \= string }                  \# required  
variable "instance\_type"     { type \= string }                  \# required  
variable "subnet\_id"         { type \= string }                  \# required  
variable "security\_group\_ids"{ type \= list(string) }            \# required  
variable "root\_volume\_size"  { type \= number, default \= 20 }    \# default  
variable "require\_imdsv2"    { type \= bool,   default \= true }  \# secure default  
variable "user\_data"         { type \= string, default \= null, sensitive \= true }  
variable "tags"              { type \= map(string), default \= {} }

## **Step 3 – Write the resources**

Reference only var., local. and resources inside the module. Never reference module.something from the root or hard-code account-specific values.

## **Step 4 – Expose outputs**

Output every attribute a caller could plausibly need: id, arn, name, computed values. Outputs are free; missing outputs force callers to hack around the module.

## **Step 5 – Add versions.tf, no provider block**

terraform {  
  required\_version \= "\>= 1.6.0"  
  required\_providers {  
    aws \= { source \= "hashicorp/aws", version \= "\~\> 6.0" }  
  }  
}

| GOTCHA A provider "aws" { region \= ... } block inside a child module is a *legacy* pattern. It prevents count/for\_each on the module and makes it non-reusable. The root declares providers; children inherit them. |
| :---- |

## **Step 6 – Document and test**

Add a README.md (inputs, outputs, example). Tools like terraform-docs generate it from your variable descriptions. Add a tests/ folder with .tftest.hcl files (Chapter 23).

## **Calling a module — every meta-argument**

module "kafka\_launch\_template" {  
  source  \= "../../modules/launch\_template"   \# required  
  version \= "1.2.0"                            \# registry/git modules only  
  count   \= var.kafka.enabled ? 1 : 0          \# or for\_each  
  providers \= { aws \= aws.west }               \# only when using an aliased provider  
  depends\_on \= \[module.network\]                \# rarely needed; prefer output references  
   
  name          \= "${var.project\_name}-kafka"  \# module inputs  
  image\_id      \= local.ami\_id  
  ...  
}

## **Module composition patterns**

**Base \+ wrapper.** launch\_template (base) is called by the apps stack for each service. A future modules/ec2\_service could wrap launch\_template \+ ec2\_instance \+ elastic\_ip into one call — a *wrapper module*.

**Facade for a stack.** The network stack is a facade: it hides the VPC, SG and IAM modules behind one tfvars file and one set of outputs.

**Passing data between modules.** Always via outputs → inputs:

module "security\_groups" {  
  vpc\_id \= module.vpc.vpc\_id      \# output of one module into input of another  
}

Terraform infers the ordering. Do not use depends\_on between modules unless there is truly no data flowing.

## **Module versioning and sources**

| Source | Pin with | Example |
| :---- | :---- | :---- |
| Local path | (git commit) | "../../modules/vpc" |
| Terraform Registry | version | source \= "terraform-aws-modules/vpc/aws", version \= "\~\> 5.0" |
| GitHub | ?ref= | "git::https://github.com/org/modules.git//vpc?ref=v1.4.0" |
| S3 / GCS bucket | object key | "s3::https://s3.amazonaws.com/bucket/vpc.zip" |

Run terraform init \-upgrade to pick up newer module versions within your constraints; terraform get refreshes local module references.

## **Refactoring resources into modules**

Moving aws\_vpc.main from the root into module.vpc without recreating it:

moved {  
  from \= aws\_vpc.main  
  to   \= module.vpc.aws\_vpc.this  
}  
moved {  
  from \= aws\_subnet.public  
  to   \= module.vpc.aws\_subnet.public\["public-a"\]  
}

Apply once, then delete the moved blocks. This is the safe way to do what the README's migration table describes.

## **Providers and modules**

**Same provider, different region** (e.g. a DR copy):

provider "aws" { region \= "us-east-1" }  
provider "aws" { alias \= "west", region \= "us-west-2" }  
   
module "vpc\_west" {  
  source    \= "../../modules/vpc"  
  providers \= { aws \= aws.west }  
  ...  
}

The module's versions.tf should declare configuration\_aliases \= \[aws\] only if it needs *multiple* providers itself; for the common case, nothing extra is needed.

## **Module design checklist**

* One responsibility

* Required inputs minimal; secure defaults for the rest

* Every input has type and description

* validation on formats

* No provider blocks, no hard-coded regions/accounts/CIDRs

* Names built from a name input; caller tags merged in

* Outputs for every ID/ARN/name

* versions.tf with constraints

* README and tests

## **Chapter Quiz**

**Q1. What is the first step when writing a module?**

a) Write outputs

b) Decide its single job

c) Pick a provider version

d) Write the README

**Q2. How should data pass from module A to module B?**

a) Shared locals

b) A's outputs into B's inputs

c) depends\_on only

d) Environment variables

**Q3. Which is legacy and blocks for\_each on a module?**

a) versions.tf in the module

b) A provider block inside the child module

c) Outputs

d) Validation

**Q4. How do you move a resource into a module without recreating it?**

a) terraform destroy then apply

b) A moved block

c) Edit the state by hand

d) Rename the tag

**Q5. What does providers \= { aws \= aws.west } do?**

a) Installs a provider

b) Passes an aliased provider configuration into the module

c) Creates a region

d) Nothing

# **Chapter 12: What to Avoid — Terraform Anti-Patterns**

Each item: the mistake, why it hurts, and what to do instead.

## **Structure and code**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| One giant main.tf with everything (the original project) | Unreadable, one state \= one blast radius | Modules \+ stacks |
| Copy-pasting resource blocks for each server | Fixes must be made N times | A module, called N times |
| Hard-coded IDs (subnet-0abc…) in code | Breaks in every other account/region | Variables, data sources, remote state |
| Hard-coded regions or account IDs | Same | var.aws\_region, data.aws\_caller\_identity |
| Values inside modules | Module is not reusable | Variables with defaults |
| type \= any everywhere | No validation, confusing errors | Precise types with optional() |
| Provider blocks in child modules | Cannot count/for\_each, not reusable | Providers in root only |
| Inline route/ingress blocks mixed with standalone aws\_route/rule resources | Perpetual diffs, rules flapping | Choose one style — standalone (as the project does) |
| count for lists of named things | Removing one item renumbers and recreates the rest | for\_each with a map |
| Deeply nested modules (module in module in module) | Hard to trace, slow plans | Two levels: base modules \+ stack |

## **State**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| Local state for a team | No locking, no sharing, easy to lose | S3 backend with locking and versioning |
| Committing terraform.tfstate to Git | Secrets in history, merge conflicts | .gitignore it; remote backend |
| Editing state JSON by hand | Corrupts state | state mv, moved, import, removed |
| \-target for routine applies | Partial state, hidden dependencies | Full applies; split into stacks if too big |
| Two people applying at once | Corruption | Locking; CI as the single applier |
| One state for everything | A typo can destroy prod networking | Stacks per blast radius, per environment |

## **Security**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| Access keys in .tf or tfvars | Leaked in Git forever | CLI profiles, SSO, CI OIDC roles |
| Passwords in tfvars committed to Git | Same | random\_password, Secrets Manager, TF\_VAR\_ in CI secrets |
| 0.0.0.0/0 on SSH/admin ports | Bots find it within minutes | allowed\_cidr \= YOUR\_IP/32, SSM instead of SSH |
| IMDSv1 (http\_tokens \= "optional") | Credential theft via SSRF | required |
| Unencrypted EBS/RDS/S3 | Compliance failures | encrypted \= true (our default) |
| AdministratorAccess on instance roles | Any compromise \= full account | Least privilege |
| Printing sensitive outputs in CI logs | Leak | sensitive \= true, mask in CI |

## **Plans, applies and drift**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| terraform apply \-auto-approve without reading a plan | Surprise deletions | Read plans; \-out then apply the file |
| Ignoring "forces replacement" | Data loss | Understand why; use create\_before\_destroy, ignore\_changes, or accept it |
| timestamp(), uuid() in resource arguments | Diff on every plan | Static values or ignore\_changes |
| Unpinned providers/modules (version missing) | Breaks randomly when upstream changes | \~\> constraints, lock file committed |
| Manual console changes to Terraform-managed resources | Drift; next apply reverts them | Change the code, or import |
| Data sources that change over time (latest AMI) in production | Unplanned instance replacement | Pin ami\_id |

## **Scripts and user data**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| sleep 60 to "wait" for a service | Flaky | Poll for readiness (our loops) |
| Scripts without set \-euo pipefail | Silent partial failures | Fail fast, log everything |
| Assuming user data re-runs on change | It doesn't | Replace instances or use SSM Run Command |
| Provisioners (remote-exec) | Unplannable, non-idempotent | User data, SSM, config management |
| Secrets baked into AMIs or user data | Readable by anyone with instance access | Fetch from Secrets Manager at boot |

## **Naming and tags**

| Avoid | Why it hurts | Instead |
| :---- | :---- | :---- |
| name on resources that get replaced (SGs, launch templates) | "already exists" errors during replacement | name\_prefix \+ create\_before\_destroy |
| Untagged resources | Mystery costs, no ownership | default\_tags \+ Name tags |
| Meaningless local names (aws\_vpc.vpc1) | Hard to read | this for singletons, role-based names otherwise |

## **Reading error messages**

Terraform's errors are precise. Read the **first** error, the file and line, and the "on ... line N, in resource ..." pointer. Common ones:

| Message | Meaning |
| :---- | :---- |
| "Unsupported argument" | Typo, or the argument does not exist for this resource/version |
| "Missing required argument" | Add it (check the docs) |
| "Invalid for\_each argument… depends on resource attributes that cannot be determined until apply" | Use a static map/set, or apply in two steps |
| "Cycle: …" | A → B → A dependency loop; split as in the SG module |
| "Error acquiring the state lock" | Someone else is applying, or a crashed run left a lock (force-unlock only if sure) |
| "DependencyViolation" | AWS refuses to delete something still in use — wrong destroy order across stacks |
| "Output refers to sensitive values" | Mark the output sensitive |
| "Inconsistent conditional result types" | Both ? : branches must match types |

## **Chapter Quiz**

**Q1. Why prefer for\_each over count for named things?**

a) It is faster

b) Removing one item does not renumber and recreate the rest

c) count is deprecated

d) for\_each supports tags

**Q2. What is the safe response to a "Cycle" error?**

a) Add depends\_on

b) Split the resources so the cycle breaks (e.g. rules separate from groups)

c) Use \-target

d) Delete state

**Q3. Why is \-auto-approve risky?**

a) It is slow

b) You never see the plan before destructive changes

c) It disables locking

d) It skips validation

**Q4. What happens to a manual console change on a Terraform-managed resource?**

a) Terraform adopts it

b) It is reported as drift and reverted on the next apply

c) Nothing

d) It becomes a variable

**Q5. "DependencyViolation" during destroy usually means:**

a) A syntax error

b) Something outside this stack still uses the resource — wrong order

c) A provider bug

d) State corruption

# **Chapter 13: Project Tour**

## **What we are building**

                 Internet  
                    |  
             \[Internet Gateway\]  
                    |  
   \+--------------- VPC 10.40.0.0/16 \----------------+  
   |                                                 |  
   |   \+-------- public-a 10.40.1.0/24 \---------+    |  
   |   |                                        |    |  
   |   |  \[Keycloak EC2\] \<---8443---- \[Kafka EC2\]|    |  
   |   |   EIP  t3.small     (private) EIP t3.med|    |  
   |   |   :8443 :9000               :8080       |    |  
   |   \+----------------------------------------+    |  
   \+-------------------------------------------------+  
        ^ your laptop: 8443 (Keycloak), 8080 (Kafka UI)

* **Keycloak** – identity server. Runs in Docker with a self-signed HTTPS certificate. Imports a realm containing the kafka-ui client and a demo user.

* **Kafka** – a single-node message broker in KRaft mode (no ZooKeeper). Only reachable inside its own Docker network.

* **Kafka UI** – a web app on port 8080\. When you open it, it redirects your browser to Keycloak to log in, then verifies the token by calling Keycloak over the private VPC address.

## **Folder layout**

kafka-keycloak-terraform/  
├── modules/  
│   ├── vpc/                 VPC, IGW, subnets, route tables, optional NAT  
│   ├── security\_groups/     map-driven SGs; rules reference each other by key  
│   ├── iam\_instance\_role/   role \+ policies \+ instance profile  
│   ├── elastic\_ip/          one EIP  
│   ├── launch\_template/     hardened base launch template  
│   └── ec2\_instance/        instance from a launch template \+ EIP association  
├── stacks/  
│   ├── 10-network/          PROCESS 1: VPC, subnets, SGs, IAM  
│   │   ├── main.tf variables.tf outputs.tf providers.tf versions.tf  
│   │   └── dev.tfvars  
│   └── 20-apps/             PROCESS 2: Keycloak \+ Kafka/Kafka UI  
│       ├── main.tf locals.tf data.tf variables.tf outputs.tf providers.tf versions.tf  
│       ├── files/           docker-compose, keycloak realm, kafka-ui config templates  
│       ├── templates/       user-data shell scripts  
│       └── dev.tfvars  
├── Makefile  
└── README.md

## **Why two processes?**

| REAL-WORLD ANALOGY When a school is built, the foundation, walls, plumbing and electrical go in first and rarely change. The furniture, computers and whiteboards change every year. You would not tear out a wall to replace a desk. The network stack is the building; the apps stack is the furniture. |
| :---- |

Practical benefits:

* Applying the apps stack cannot touch the VPC, even by accident.

* The network can be built once by a platform team and reused by many app stacks.

* Smaller plans are faster to run and easier to review.

* Different backends, permissions and change-approval rules per stack.

## **How the stacks talk**

The network stack ends with output blocks. The apps stack reads them with a terraform\_remote\_state data source, then looks things up by key: public\_subnet\_ids\["public-a"\], security\_group\_ids\["keycloak"\], instance\_profile\_names\["ec2\_ssm"\]. The keys are set in dev.tfvars, so you can rename or add subnets without touching code. If you already have a VPC, pass the raw IDs instead and the remote state read is skipped.

## **Reading order**

The chapters that follow walk through every file: modules first (the building blocks), then the network stack (process 1), then the apps stack (process 2), then deployment and verification. Each code listing is followed by a line-by-line table explaining **what** the line does, **why** it exists, and **how** it works.

## **Chapter Quiz**

**Q1. Which server does your browser talk to on port 8080?**

a) Keycloak

b) Kafka broker

c) Kafka UI

d) The Internet Gateway

**Q2. How does Kafka UI verify a login token?**

a) It asks Kafka

b) It calls Keycloak over the private VPC address on 8443

c) It checks a local file

d) It asks your laptop

**Q3. The 10- and 20- prefixes on stack folders indicate:**

a) Version numbers

b) Run order

c) Port numbers

d) Nothing

**Q4. Where do compose files and startup scripts live?**

a) modules/

b) stacks/10-network

c) stacks/20-apps/files and templates

d) The AWS console

**Q5. What passes information from the network stack to the apps stack?**

a) Environment variables

b) Outputs read through remote state

c) Copying IDs by hand

d) Tags

# **Chapter 14: The VPC Module, Line by Line**

Folder: modules/vpc/. This module is the "building": VPC, front door (IGW), rooms (subnets), and exit signs (route tables).

## **variables.tf**

variable "name"                    { type \= string }  
variable "cidr\_block"              { type \= string }  
variable "enable\_dns\_support"      { type \= bool, default \= true }  
variable "enable\_dns\_hostnames"    { type \= bool, default \= true }  
variable "create\_internet\_gateway" { type \= bool, default \= true }  
variable "default\_availability\_zone" { type \= string, default \= null }  
   
variable "public\_subnets" {  
  type \= map(object({  
    cidr\_block              \= string  
    availability\_zone       \= optional(string)  
    map\_public\_ip\_on\_launch \= optional(bool, true)  
    tags                    \= optional(map(string), {})  
  }))  
  default \= {}  
}  
   
variable "private\_subnets" {  
  type \= map(object({  
    cidr\_block        \= string  
    availability\_zone \= optional(string)  
    tags              \= optional(map(string), {})  
  }))  
  default \= {}  
}  
   
variable "create\_nat\_gateway" { type \= bool, default \= false }  
variable "tags"               { type \= map(string), default \= {} }

| Variable | What | Why |
| :---- | :---- | :---- |
| name | Prefix for every Name tag | Lets you tell projects apart in the console |
| cidr\_block | VPC address range | Required by AWS |
| enable\_dns\_\* | Turn on AWS's internal DNS | Needed so instances get hostnames; SSM and many services expect it |
| create\_internet\_gateway | Build the front door? | A fully private VPC would set this false |
| default\_availability\_zone | Fallback AZ | Subnets that don't pin an AZ use this; the network stack computes it |
| public\_subnets | Map of subnets | A map means "as many as you like, each with a name" |
| map\_public\_ip\_on\_launch | Auto-assign public IPs | True for public subnets so instances can reach the internet immediately |
| private\_subnets | Same idea, no internet route | Optional; empty by default |
| create\_nat\_gateway | Mail slot for private subnets | Off by default because NAT costs money (\~$32/month) |
| tags | Extra labels | Cost tracking, ownership |

## **main.tf – the VPC and gateway**

resource "aws\_vpc" "this" {  
  cidr\_block           \= var.cidr\_block  
  enable\_dns\_support   \= var.enable\_dns\_support  
  enable\_dns\_hostnames \= var.enable\_dns\_hostnames  
  tags \= merge(var.tags, { Name \= "${var.name}-vpc" })  
}  
   
resource "aws\_internet\_gateway" "this" {  
  count  \= var.create\_internet\_gateway ? 1 : 0  
  vpc\_id \= aws\_vpc.this.id  
  tags   \= merge(var.tags, { Name \= "${var.name}-igw" })  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| resource "aws\_vpc" "this" | Create the VPC | this is a common name for "the one thing this module makes" |
| cidr\_block \= var.cidr\_block | Address range | Comes from the caller |
| enable\_dns\_hostnames | Give instances DNS names | Without it, ip-10-40-1-5.ec2.internal names don't resolve |
| merge(var.tags, {Name=...}) | Combine caller tags with a Name | merge lets caller tags apply everywhere while the module controls Name |
| count \= ... ? 1 : 0 | Optional IGW | Terraform idiom: count of 0 means "don't create" |
| vpc\_id \= aws\_vpc.this.id | Attach IGW to VPC | Implicit dependency: VPC first |

## **main.tf – public subnets**

resource "aws\_subnet" "public" {  
  for\_each \= var.public\_subnets  
   
  vpc\_id                  \= aws\_vpc.this.id  
  cidr\_block              \= each.value.cidr\_block  
  availability\_zone       \= coalesce(each.value.availability\_zone, var.default\_availability\_zone)  
  map\_public\_ip\_on\_launch \= each.value.map\_public\_ip\_on\_launch  
   
  lifecycle {  
    precondition {  
      condition     \= coalesce(each.value.availability\_zone, var.default\_availability\_zone, "unset") \!= "unset"  
      error\_message \= "Subnet '${each.key}' has no availability\_zone and no default\_availability\_zone was supplied."  
    }  
  }  
   
  tags \= merge(var.tags, each.value.tags, { Name \= "${var.name}-${each.key}", Tier \= "public" })  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| for\_each \= var.public\_subnets | One subnet per map entry | Key becomes part of the address: aws\_subnet.public\["public-a"\] |
| each.value.cidr\_block | This subnet's range | each.value is the object for the current key |
| coalesce(a, b) | Per-subnet AZ, else default | First non-null wins |
| precondition | Fail early with a clear message | Better than an AWS error about a null AZ |
| Tier \= "public" | Extra tag | Tools and humans can filter public vs private |

resource "aws\_route\_table" "public" {  
  count  \= length(var.public\_subnets) \> 0 ? 1 : 0  
  vpc\_id \= aws\_vpc.this.id  
  tags   \= merge(var.tags, { Name \= "${var.name}-public-rt" })  
}  
   
resource "aws\_route" "public\_internet" {  
  count                  \= length(var.public\_subnets) \> 0 && var.create\_internet\_gateway ? 1 : 0  
  route\_table\_id         \= aws\_route\_table.public\[0\].id  
  destination\_cidr\_block \= "0.0.0.0/0"  
  gateway\_id             \= aws\_internet\_gateway.this\[0\].id  
}  
   
resource "aws\_route\_table\_association" "public" {  
  for\_each       \= aws\_subnet.public  
  subnet\_id      \= each.value.id  
  route\_table\_id \= aws\_route\_table.public\[0\].id  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| aws\_route\_table with count | One shared table for all public subnets | Only if there are public subnets |
| aws\_route as a separate resource | The 0.0.0.0/0 → IGW route | Separate (not inline) so it can be conditional on the IGW existing |
| destination\_cidr\_block \= "0.0.0.0/0" | "Everywhere else" | The default route |
| for\_each \= aws\_subnet.public | One association per subnet | You can for\_each over a resource's instances directly |

| GOTCHA Never mix inline route { } blocks inside aws\_route\_table with separate aws\_route resources for the same table. They fight each other and Terraform will show changes forever. |
| :---- |

## **main.tf – private subnets and NAT**

The private block mirrors the public one, with a NAT Gateway instead of an IGW route:

resource "aws\_nat\_gateway" "this" {  
  count         \= var.create\_nat\_gateway && length(var.private\_subnets) \> 0 ? 1 : 0  
  allocation\_id \= aws\_eip.nat\[0\].id  
  subnet\_id     \= values(aws\_subnet.public)\[0\].id  
  depends\_on    \= \[aws\_internet\_gateway.this\]  
  lifecycle {  
    precondition {  
      condition     \= length(var.public\_subnets) \> 0  
      error\_message \= "create\_nat\_gateway requires at least one public subnet."  
    }  
  }  
}

| Line | Why |
| :---- | :---- |
| allocation\_id \= aws\_eip.nat\[0\].id | A NAT Gateway needs its own Elastic IP |
| values(aws\_subnet.public)\[0\].id | NAT must sit in a *public* subnet (it needs the IGW) |
| depends\_on \= \[aws\_internet\_gateway.this\] | AWS docs require the IGW to exist first; no attribute reference exists, so we say it explicitly |

## **outputs.tf**

output "vpc\_id"             { value \= aws\_vpc.this.id }  
output "public\_subnet\_ids"  { value \= { for k, s in aws\_subnet.public : k \=\> s.id } }  
output "private\_subnet\_ids" { value \= { for k, s in aws\_subnet.private : k \=\> s.id } }  
output "public\_subnet\_azs"  { value \= { for k, s in aws\_subnet.public : k \=\> s.availability\_zone } }  
output "internet\_gateway\_id" { value \= try(aws\_internet\_gateway.this\[0\].id, null) }

The for expressions turn the resource map into a plain key \=\> id map, which is exactly what callers want: module.vpc.public\_subnet\_ids\["public-a"\].

## **Verify with the AWS CLI**

VPC=$(terraform \-chdir=stacks/10-network output \-raw vpc\_id)  
aws ec2 describe-vpcs \--vpc-ids $VPC  
aws ec2 describe-internet-gateways \--filters Name=attachment.vpc-id,Values=$VPC  
aws ec2 describe-subnets \--filters Name=vpc-id,Values=$VPC \--output table  
aws ec2 describe-route-tables \--filters Name=vpc-id,Values=$VPC \\  
  \--query 'RouteTables\[\].{Id:RouteTableId,Routes:Routes\[\].{Dest:DestinationCidrBlock,Via:GatewayId}}'

| OFFICIAL DOCS aws\_vpc: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc · aws\_subnet: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet · aws\_route: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route |
| :---- |

## **Chapter Quiz**

**Q1. Why is aws\_route a separate resource instead of an inline route block?**

a) It is faster

b) So it can be conditional and avoid inline/separate conflicts

c) AWS requires it

d) For tagging

**Q2. What does for\_each \= aws\_subnet.public on the route table association do?**

a) Creates one association for each subnet the module created

b) Creates one subnet

c) Errors

d) Associates all subnets to all tables

**Q3. Where must a NAT Gateway be placed?**

a) In a private subnet

b) In a public subnet

c) Outside the VPC

d) Anywhere

**Q4. What does coalesce(each.value.availability\_zone, var.default\_availability\_zone) return?**

a) Both values

b) The first non-null value

c) An error if both are set

d) null always

**Q5. Why is enable\_dns\_hostnames true?**

a) For faster networking

b) So instances get resolvable DNS names, which SSM and services expect

c) To save money

d) It is required by Terraform

# **Chapter 15: The Security Groups Module, Line by Line**

Folder: modules/security\_groups/. This is the "hall monitor" module. It takes a map of groups and rules from tfvars and builds them all.

## **The design problem it solves**

The Keycloak group needs a rule "allow port 8443 from the Kafka group." If you wrote both groups in one for\_each with inline rules, Terraform would see a cycle (Keycloak needs Kafka's ID, which is created in the same loop). The fix: create all groups first with **no rules**, then create every rule as its own resource that references groups by ID. Since the rules are separate resources, they can look up any group by key.

| REAL-WORLD ANALOGY First hire all the hall monitors and give them name tags. Then hand out the rule cards, which can say "let in anyone sent by Monitor Kafka." You could not write that card before Monitor Kafka existed. |
| :---- |

## **variables.tf (the important part)**

variable "security\_groups" {  
  type \= map(object({  
    description \= string  
    tags        \= optional(map(string), {})  
    ingress \= optional(list(object({  
      description               \= optional(string)  
      from\_port                 \= optional(number)  
      to\_port                   \= optional(number)  
      protocol                  \= optional(string, "tcp")  
      cidr\_ipv4                 \= optional(string)  
      cidr\_ipv6                 \= optional(string)  
      source\_security\_group\_key \= optional(string)  
      source\_security\_group\_id  \= optional(string)  
      self                      \= optional(bool, false)  
    })), \[\])  
    egress \= optional(list(object({ ...same fields..., protocol \= optional(string, "-1") })),  
      \[{ description \= "Allow all outbound", protocol \= "-1", cidr\_ipv4 \= "0.0.0.0/0" }\])  
  }))  
  validation {  
    condition \= alltrue(flatten(\[  
      for sg in var.security\_groups : \[  
        for r in concat(sg.ingress, sg.egress) :  
        length(\[for v in \[r.cidr\_ipv4, r.cidr\_ipv6, r.source\_security\_group\_key,  
                          r.source\_security\_group\_id, r.self ? "self" : null\] : v if v \!= null\]) \== 1  
      \]  
    \]))  
    error\_message \= "Every rule must set exactly one of cidr\_ipv4, cidr\_ipv6, source\_security\_group\_key, source\_security\_group\_id or self."  
  }  
}

| Field | Meaning |
| :---- | :---- |
| cidr\_ipv4 | Allow from an IP range |
| source\_security\_group\_key | Allow from another group *in this same map*, by key |
| source\_security\_group\_id | Allow from an existing group outside this module |
| self | Allow from members of this same group |
| protocol \= "-1" | All protocols; ports are ignored |
| egress default | If you say nothing, outbound is wide open (typical for servers that must download packages) |

The validation counts how many source fields are set on each rule and insists on exactly one. This catches tfvars typos before AWS sees them.

## **main.tf – groups first**

resource "aws\_security\_group" "this" {  
  for\_each    \= var.security\_groups  
  name\_prefix \= "${var.name}-${each.key}-"  
  description \= each.value.description  
  vpc\_id      \= var.vpc\_id  
  tags        \= merge(var.tags, each.value.tags, { Name \= "${var.name}-${each.key}-sg" })  
  lifecycle { create\_before\_destroy \= true }  
}

| Line | Why |
| :---- | :---- |
| name\_prefix instead of name | AWS appends a random suffix, so a replacement can be created before the old one is deleted |
| create\_before\_destroy \= true | If the group must be replaced, the new one exists before the old is removed, so instances are never left with no firewall |
| No inline ingress/egress | Rules come next as separate resources |

## **main.tf – flatten the rules**

locals {  
  ingress\_rules \= flatten(\[  
    for sg\_key, sg in var.security\_groups : \[  
      for idx, rule in sg.ingress : {  
        id     \= "${sg\_key}-ingress-${idx}"  
        sg\_key \= sg\_key  
        ...all rule fields...  
      }  
    \]  
  \])  
}

Two nested fors produce a list of lists (one inner list per group). flatten turns that into one flat list. Each entry gets a unique id like keycloak-ingress-1 which becomes the for\_each key.

| GOTCHA The key includes the list index, so reordering rules in tfvars changes keys and Terraform will delete and recreate the affected rules. Harmless for security groups (a few seconds), but be aware. |
| :---- |

## **main.tf – the rule resources**

resource "aws\_vpc\_security\_group\_ingress\_rule" "this" {  
  for\_each          \= { for r in local.ingress\_rules : r.id \=\> r }  
  security\_group\_id \= aws\_security\_group.this\[each.value.sg\_key\].id  
  description       \= each.value.description  
  ip\_protocol       \= each.value.protocol  
  from\_port         \= each.value.protocol \== "-1" ? null : each.value.from\_port  
  to\_port           \= each.value.protocol \== "-1" ? null : each.value.to\_port  
  cidr\_ipv4         \= each.value.cidr\_ipv4  
  cidr\_ipv6         \= each.value.cidr\_ipv6  
  referenced\_security\_group\_id \= (  
    each.value.self ? aws\_security\_group.this\[each.value.sg\_key\].id :  
    each.value.source\_security\_group\_key \!= null ? aws\_security\_group.this\[each.value.source\_security\_group\_key\].id :  
    each.value.source\_security\_group\_id  
  )  
  tags \= merge(var.tags, { Name \= each.key })  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| { for r in ... : r.id \=\> r } | List → map keyed by id | for\_each needs a map or set |
| aws\_security\_group.this\[each.value.sg\_key\].id | Which group this rule belongs to | Looks up the group by key |
| ip\_protocol / ports null when \-1 | All-traffic rules must not set ports | AWS rejects ports with protocol \-1 |
| Nested conditional for referenced\_security\_group\_id | Resolve self / key / raw id | Only one is set (guaranteed by the validation) |
| aws\_vpc\_security\_group\_ingress\_rule (new-style) | One resource per rule | Newer than inline rules; each rule gets its own ID and tags, and changes are surgical |

The egress resource is identical with aws\_vpc\_security\_group\_egress\_rule.

## **Verify with the AWS CLI**

SG=$(terraform \-chdir=stacks/10-network output \-json security\_group\_ids | jq \-r .keycloak)  
aws ec2 describe-security-groups \--group-ids $SG \--query 'SecurityGroups\[0\].IpPermissions'  
aws ec2 describe-security-group-rules \--filters Name=group-id,Values=$SG \--output table

Look for a rule whose ReferencedGroupInfo.GroupId is the Kafka group — that is the SG-to-SG rule.

| OFFICIAL DOCS https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc\_security\_group\_ingress\_rule · Security group rules reference: https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html |
| :---- |

## **Chapter Quiz**

**Q1. Why are groups created without rules first?**

a) Speed

b) To avoid a dependency cycle when rules reference other groups

c) AWS requires it

d) Rules are optional

**Q2. What does flatten do to a list of lists?**

a) Sorts it

b) Removes duplicates

c) Makes one flat list

d) Reverses it

**Q3. What happens if a rule sets both cidr\_ipv4 and source\_security\_group\_key?**

a) Both apply

b) The validation fails with a clear message

c) The first wins

d) AWS decides

**Q4. Why create\_before\_destroy on security groups?**

a) Cheaper

b) So a replacement exists before the old one is removed and instances keep a firewall

c) Required by for\_each

d) For tags

**Q5. With protocol \-1, what must the ports be?**

a) 0

b) 65535

c) null / unset

d) 22

# **Chapter 16: The IAM Role and Elastic IP Modules**

## **IAM instance role — modules/iam\_instance\_role/**

data "aws\_iam\_policy\_document" "assume" {  
  statement {  
    effect  \= "Allow"  
    actions \= \["sts:AssumeRole"\]  
    principals {  
      type        \= "Service"  
      identifiers \= \["ec2.amazonaws.com"\]  
    }  
  }  
}  
   
resource "aws\_iam\_role" "this" {  
  name\_prefix        \= "${var.name}-"  
  description        \= var.description  
  assume\_role\_policy \= data.aws\_iam\_policy\_document.assume.json  
  tags               \= merge(var.tags, { Name \= "${var.name}-role" })  
}  
   
resource "aws\_iam\_role\_policy\_attachment" "managed" {  
  for\_each   \= toset(var.managed\_policy\_arns)  
  role       \= aws\_iam\_role.this.name  
  policy\_arn \= each.value  
}  
   
resource "aws\_iam\_role\_policy" "inline" {  
  for\_each \= var.inline\_policies  
  name     \= each.key  
  role     \= aws\_iam\_role.this.id  
  policy   \= each.value  
}  
   
resource "aws\_iam\_instance\_profile" "this" {  
  name\_prefix \= "${var.name}-"  
  role        \= aws\_iam\_role.this.name  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| data "aws\_iam\_policy\_document" | Builds JSON for the trust policy | Safer than hand-writing JSON; Terraform validates the structure |
| principals { Service \= ec2.amazonaws.com } | "EC2 may assume this role" | This is the **trust policy** — who may wear the badge |
| aws\_iam\_role | The badge itself |  |
| for\_each \= toset(var.managed\_policy\_arns) | Attach each managed policy | toset because for\_each over a list needs a set |
| AmazonSSMManagedInstanceCore (default) | Permissions the SSM agent needs | Enables Session Manager without SSH |
| aws\_iam\_role\_policy inline | Optional custom policies | Map of name → JSON |
| aws\_iam\_instance\_profile | Holder for the role | EC2 attaches profiles, not roles |

| NOTE Two policies are involved in every role. The trust policy says *who can assume* the role (EC2). The permission policies say *what the role can do* (talk to SSM). Beginners often confuse them. |
| :---- |

| REAL-WORLD ANALOGY The trust policy is the sign on the key cabinet: "Substitute teachers may take a key." The permission policy is what the key opens. |
| :---- |

Outputs: role\_name, role\_arn, instance\_profile\_name, instance\_profile\_arn, policy\_attachment\_ids.

## **Elastic IP — modules/elastic\_ip/**

resource "aws\_eip" "this" {  
  domain \= "vpc"  
  tags   \= merge(var.tags, { Name \= "${var.name}-eip" })  
}  
output "allocation\_id" { value \= aws\_eip.this.id }  
output "public\_ip"     { value \= aws\_eip.this.public\_ip }

Tiny on purpose. domain \= "vpc" is required on modern accounts. The EIP is created **without** an instance so its public\_ip is known before we render user data. The association to an instance happens later in the ec2\_instance module.

| GOTCHA An allocated but unattached EIP costs a small hourly fee. Always run destroy on labs you are done with. |
| :---- |

## **Verify with the AWS CLI**

aws iam get-role \--role-name $(aws iam list-roles \--query 'Roles\[?contains(RoleName,\`ec2\_ssm\`)\].RoleName' \--output text)  
aws iam list-attached-role-policies \--role-name \<role-name\>  
aws iam list-instance-profiles \--query 'InstanceProfiles\[?contains(InstanceProfileName,\`kafka\`)\]'  
aws ec2 describe-addresses \--query 'Addresses\[\].{IP:PublicIp,Instance:InstanceId,Name:Tags\[?Key==\`Name\`\].Value|\[0\]}' \--output table

| OFFICIAL DOCS IAM roles for EC2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html · Elastic IPs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html |
| :---- |

## **Chapter Quiz**

**Q1. What is a trust policy?**

a) What the role can do

b) Who may assume the role

c) A tag

d) A password

**Q2. Why use data "aws\_iam\_policy\_document" instead of a JSON string?**

a) It is shorter

b) Terraform validates structure and it composes well

c) AWS requires it

d) It is faster

**Q3. Why is toset() used with for\_each on managed\_policy\_arns?**

a) To sort

b) for\_each requires a map or set, not a list

c) To remove nulls

d) For tagging

**Q4. Why is the EIP created before the instance?**

a) EIPs are slow

b) So the public IP can be embedded in user data

c) It is cheaper

d) The instance module cannot create EIPs

**Q5. What attaches a role to an EC2 instance?**

a) The role ARN directly

b) An instance profile

c) A security group

d) A tag

# **Chapter 17: The Launch Template and EC2 Instance Modules**

## **Launch template — modules/launch\_template/**

This is the "classroom package" — a reusable recipe with secure defaults. Every argument is a variable with a sensible default.

resource "aws\_launch\_template" "this" {  
  name\_prefix            \= "${var.name}-"  
  description            \= var.description  
  image\_id               \= var.image\_id  
  instance\_type          \= var.instance\_type  
  key\_name               \= var.key\_name  
  update\_default\_version \= var.update\_default\_version  
  ebs\_optimized          \= var.ebs\_optimized  
   
  dynamic "iam\_instance\_profile" {  
    for\_each \= var.iam\_instance\_profile\_name \== null ? \[\] : \[1\]  
    content { name \= var.iam\_instance\_profile\_name }  
  }  
   
  network\_interfaces {  
    device\_index                \= 0  
    associate\_public\_ip\_address \= var.associate\_public\_ip\_address  
    delete\_on\_termination       \= true  
    subnet\_id                   \= var.subnet\_id  
    security\_groups             \= var.security\_group\_ids  
  }  
   
  user\_data \= var.user\_data \== null ? null : base64encode(var.user\_data)  
   
  block\_device\_mappings {  
    device\_name \= var.root\_device\_name  
    ebs {  
      volume\_type           \= var.root\_volume\_type  
      volume\_size           \= var.root\_volume\_size  
      iops                  \= var.root\_volume\_iops  
      throughput            \= var.root\_volume\_throughput  
      encrypted             \= var.root\_volume\_encrypted  
      kms\_key\_id            \= var.root\_volume\_kms\_key\_id  
      delete\_on\_termination \= true  
    }  
  }  
   
  dynamic "block\_device\_mappings" {  
    for\_each \= var.additional\_volumes  
    content {  
      device\_name \= block\_device\_mappings.value.device\_name  
      ebs { ... }  
    }  
  }  
   
  metadata\_options {  
    http\_endpoint               \= "enabled"  
    http\_tokens                 \= var.require\_imdsv2 ? "required" : "optional"  
    http\_put\_response\_hop\_limit \= var.metadata\_hop\_limit  
    instance\_metadata\_tags      \= var.enable\_instance\_metadata\_tags ? "enabled" : "disabled"  
  }  
   
  monitoring { enabled \= var.detailed\_monitoring }  
   
  tag\_specifications {  
    resource\_type \= "instance"  
    tags \= merge(var.tags, var.instance\_tags, { Name \= "${var.name}-ec2" })  
  }  
  tag\_specifications {  
    resource\_type \= "volume"  
    tags \= merge(var.tags, var.volume\_tags, { Name \= "${var.name}-root" })  
  }  
  tags \= merge(var.tags, { Name \= "${var.name}-launch-template" })  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| name\_prefix | Unique name with suffix | Lets Terraform create a new template before removing the old |
| image\_id | The AMI | Passed in; the apps stack resolves it from SSM |
| update\_default\_version \= true | New versions become the default | Anything launching from "default" gets the latest recipe |
| dynamic "iam\_instance\_profile" | Include the block only if a profile was given | dynamic builds a nested block from a list; empty list \= no block |
| network\_interfaces | Subnet, SG, public IP on the primary NIC | Put here rather than at instance level so the template is complete |
| user\_data \= base64encode(...) | Startup script | AWS requires base64; the module does it so callers pass plain text |
| block\_device\_mappings | Root disk | gp3 encrypted by default — security best practice at no extra cost |
| dynamic "block\_device\_mappings" | Extra disks | e.g. a data disk for Kafka logs |
| http\_tokens \= "required" | IMDSv2 only | Blocks a whole class of credential-theft attacks (SSRF) |
| http\_put\_response\_hop\_limit | How far metadata replies may travel | Default 1; set 2 if Docker containers must reach IMDS |
| monitoring | 1-minute CloudWatch metrics | Off by default (costs a little) |
| tag\_specifications × 2 | Tags for instance and volume | Volumes are separate resources; without this they are untagged |

| GOTCHA user\_data in a launch template is *not* re-run when you change it on an existing instance. A new template version is created, but the instance keeps running the old script until it is replaced. To roll out new user data, replace the instance (terraform taint / \-replace). |
| :---- |

| NOTE The user\_data variable is marked sensitive \= true because our scripts embed passwords. Terraform hides it in plans. |
| :---- |

## **EC2 instance — modules/ec2\_instance/**

resource "aws\_instance" "this" {  
  launch\_template {  
    id      \= var.launch\_template\_id  
    version \= var.launch\_template\_version  
  }  
  disable\_api\_termination \= var.termination\_protection  
  tags \= merge(var.tags, { Name \= "${var.name}-ec2" })  
}  
   
resource "aws\_eip\_association" "this" {  
  count         \= var.eip\_allocation\_id \== null ? 0 : 1  
  instance\_id   \= aws\_instance.this.id  
  allocation\_id \= var.eip\_allocation\_id  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| launch\_template { id, version } | Build from the recipe | No AMI/type/subnet repeated here — the template owns them |
| version \= tostring(latest\_version) | Pin to the exact version | Changing the template bumps the version, which changes this argument, which replaces the instance — exactly what we want for a new user-data script |
| disable\_api\_termination | Guard against accidental delete | Off for labs, on for production |
| aws\_eip\_association with count | Optional EIP | Attaching happens *after* the instance exists |

Outputs: id, arn, private\_ip, public\_ip (the EIP if attached), availability\_zone.

## **Verify with the AWS CLI**

aws ec2 describe-launch-templates \--query 'LaunchTemplates\[\].{Name:LaunchTemplateName,Latest:LatestVersionNumber,Default:DefaultVersionNumber}' \--output table  
aws ec2 describe-launch-template-versions \--launch-template-name \<name\> \--versions '$Latest' \\  
  \--query 'LaunchTemplateVersions\[0\].LaunchTemplateData.{Type:InstanceType,AMI:ImageId,IMDS:MetadataOptions.HttpTokens}'  
ID=$(terraform \-chdir=stacks/20-apps output \-raw keycloak\_instance\_id)  
aws ec2 describe-instances \--instance-ids $ID \--query 'Reservations\[0\].Instances\[0\].{State:State.Name,Private:PrivateIpAddress,Public:PublicIpAddress,Profile:IamInstanceProfile.Arn,IMDS:MetadataOptions.HttpTokens}'  
aws ec2 get-console-output \--instance-id $ID \--latest \--output text | tail \-50

| OFFICIAL DOCS Launch templates: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html · IMDSv2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html · aws\_launch\_template: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch\_template |
| :---- |

## **Chapter Quiz**

**Q1. What does a dynamic block do?**

a) Makes the resource faster

b) Generates zero or more nested blocks from a collection

c) Encrypts data

d) Creates a variable

**Q2. Why require IMDSv2?**

a) It is faster

b) It blocks credential theft via SSRF-style attacks

c) Terraform requires it

d) It is free

**Q3. Changing user data on an existing instance's launch template will:**

a) Re-run the script immediately

b) Create a new template version; the instance must be replaced to run it

c) Do nothing

d) Reboot the instance

**Q4. Why pin version \= tostring(latest\_version) on the instance?**

a) For speed

b) So template changes flow through to replace the instance

c) AWS requires it

d) To avoid replacement

**Q5. Why two tag\_specifications blocks?**

a) One is a typo

b) Instances and volumes are tagged separately

c) For the EIP

d) For the subnet

# **Chapter 18: The Network Stack (Process 1), Line by Line**

Folder: stacks/10-network/. This is the first thing you apply. It builds the foundation and publishes outputs for the apps stack.

## **versions.tf and providers.tf**

terraform {  
  required\_version \= "\>= 1.6.0"  
  required\_providers {  
    aws \= { source \= "hashicorp/aws", version \= "\~\> 6.0" }  
  }  
  \# backend "s3" { ... }   \# uncomment for shared state  
}  
   
provider "aws" {  
  region \= var.aws\_region  
  default\_tags {  
    tags \= merge({ Project \= var.project\_name, Stack \= "network", ManagedBy \= "terraform" }, var.tags)  
  }  
}

| Line | Why |
| :---- | :---- |
| \~\> 6.0 | Any 6.x, never 7.0 — protects you from breaking changes |
| backend "s3" commented | Local state for a lab; Chapter 22 shows how to turn it on |
| default\_tags | Every resource gets Project/Stack/ManagedBy tags for free — cost reports and "who made this?" answers |

## **variables.tf**

The interesting variables:

variable "availability\_zone"               { type \= string, default \= null }  
variable "instance\_types\_for\_az\_selection" { type \= list(string), default \= \["t3.medium", "t3.small"\] }  
variable "vpc\_cidr"        { type \= string, default \= "10.40.0.0/16" }  
variable "public\_subnets"  { type \= map(object({...})), default \= { public-a \= { cidr\_block \= "10.40.1.0/24" } } }  
variable "allowed\_cidr"    { type \= string, default \= "0.0.0.0/0" }  
variable "security\_groups" { type \= map(object({...})) }  
variable "instance\_roles"  { type \= map(object({ managed\_policy\_arns \= optional(list(string), \[...SSM...\]) })) }

Everything a different environment might change is here. Nothing is hard-coded in main.tf.

## **main.tf – choosing the Availability Zone**

data "aws\_ec2\_instance\_type\_offerings" "by\_type" {  
  for\_each      \= var.availability\_zone \== null ? toset(var.instance\_types\_for\_az\_selection) : toset(\[\])  
  location\_type \= "availability-zone"  
  filter {  
    name   \= "instance-type"  
    values \= \[each.value\]  
  }  
}  
   
locals {  
  az\_sets    \= \[for k, d in data.aws\_ec2\_instance\_type\_offerings.by\_type : toset(d.locations)\]  
  common\_azs \= length(local.az\_sets) \== 0 ? \[\] : sort(tolist(setintersection(local.az\_sets...)))  
  selected\_availability\_zone \= coalesce(var.availability\_zone, try(local.common\_azs\[0\], null))  
}  
   
check "availability\_zone\_available" {  
  assert {  
    condition     \= local.selected\_availability\_zone \!= null  
    error\_message \= "No AZ in ${var.aws\_region} supports all of: ${join(", ", var.instance\_types\_for\_az\_selection)}."  
  }  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| data ... for\_each \= ... | Ask AWS which AZs offer each instance type | One query per type; skipped entirely if the user pinned an AZ |
| location\_type \= "availability-zone" | Answer in AZ names | Could also answer by region |
| az\_sets | A list of sets, one per type | e.g. \[{a,b,c}, {a,b,d}\] |
| setintersection(local.az\_sets...) | AZs in *every* set | The ... spreads a list into separate arguments |
| sort(tolist(...)) | Deterministic order | Sets are unordered; sorting means the same AZ is chosen every run |
| coalesce(var.availability\_zone, try(...)) | Manual pin wins, else first common AZ |  |
| check block | Friendly warning if nothing matches |  |

| REAL-WORLD ANALOGY Before choosing which store to open the bakery in, you check which locations have both the ovens and the mixers you need. You pick the first one alphabetically so everyone lands on the same answer. |
| :---- |

## **main.tf – the ALLOWED\_CIDR trick**

locals {  
  security\_groups \= {  
    for k, sg in var.security\_groups : k \=\> merge(sg, {  
      ingress \= \[for r in sg.ingress : merge(r, { cidr\_ipv4 \= r.cidr\_ipv4 \== "ALLOWED\_CIDR" ? var.allowed\_cidr : r.cidr\_ipv4 })\]  
      egress  \= \[for r in sg.egress  : merge(r, { cidr\_ipv4 \= r.cidr\_ipv4 \== "ALLOWED\_CIDR" ? var.allowed\_cidr : r.cidr\_ipv4 })\]  
    })  
  }  
}

In tfvars you write cidr\_ipv4 \= "ALLOWED\_CIDR" on three rules. This local swaps the placeholder for the real value of var.allowed\_cidr. Change your IP once, and every rule updates.

## **main.tf – calling the modules**

module "vpc" {  
  source                    \= "../../modules/vpc"  
  name                      \= var.project\_name  
  cidr\_block                \= var.vpc\_cidr  
  default\_availability\_zone \= local.selected\_availability\_zone  
  public\_subnets            \= var.public\_subnets  
  private\_subnets           \= var.private\_subnets  
  create\_nat\_gateway        \= var.create\_nat\_gateway  
  tags                      \= var.tags  
}  
   
module "security\_groups" {  
  source          \= "../../modules/security\_groups"  
  name            \= var.project\_name  
  vpc\_id          \= module.vpc.vpc\_id  
  security\_groups \= local.security\_groups  
  tags            \= var.tags  
}  
   
module "instance\_roles" {  
  source              \= "../../modules/iam\_instance\_role"  
  for\_each            \= var.instance\_roles  
  name                \= "${var.project\_name}-${each.key}"  
  description         \= each.value.description  
  managed\_policy\_arns \= each.value.managed\_policy\_arns  
  inline\_policies     \= each.value.inline\_policies  
  tags                \= var.tags  
}

Notice how thin this is. The root module only wires variables to modules. vpc\_id \= module.vpc.vpc\_id creates the dependency: security groups wait for the VPC.

## **outputs.tf**

output "vpc\_id"                 { value \= module.vpc.vpc\_id }  
output "public\_subnet\_ids"      { value \= module.vpc.public\_subnet\_ids }  
output "security\_group\_ids"     { value \= module.security\_groups.security\_group\_ids }  
output "instance\_profile\_names" { value \= { for k, m in module.instance\_roles : k \=\> m.instance\_profile\_name } }  
output "selected\_availability\_zone" { value \= local.selected\_availability\_zone }

These are the **contract** with the apps stack. Add outputs freely; never rename them without updating consumers.

## **dev.tfvars – the order form**

aws\_region   \= "us-east-1"  
project\_name \= "kafka-keycloak-lab"  
tags         \= { Environment \= "dev" }  
   
availability\_zone               \= null  
instance\_types\_for\_az\_selection \= \["t3.medium", "t3.small"\]  
   
vpc\_cidr \= "10.40.0.0/16"  
public\_subnets \= {  
  public-a \= { cidr\_block \= "10.40.1.0/24" }  
}  
   
allowed\_cidr \= "0.0.0.0/0"     \# change to YOUR\_IP/32  
   
security\_groups \= {  
  kafka \= {  
    description \= "Kafka and Kafka UI EC2 access"  
    ingress \= \[  
      { description \= "Kafka UI", from\_port \= 8080, to\_port \= 8080, cidr\_ipv4 \= "ALLOWED\_CIDR" }  
    \]  
  }  
  keycloak \= {  
    description \= "Keycloak EC2 access"  
    ingress \= \[  
      { description \= "Keycloak HTTPS", from\_port \= 8443, to\_port \= 8443, cidr\_ipv4 \= "ALLOWED\_CIDR" },  
      { description \= "Kafka UI back-channel", from\_port \= 8443, to\_port \= 8443, source\_security\_group\_key \= "kafka" },  
      { description \= "Health and metrics", from\_port \= 9000, to\_port \= 9000, cidr\_ipv4 \= "ALLOWED\_CIDR" }  
    \]  
  }  
}  
   
instance\_roles \= {  
  ec2\_ssm \= { managed\_policy\_arns \= \["arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"\] }  
}

| BEST PRACTICE Set allowed\_cidr to your own IP with /32. Find it with curl \-s https://checkip.amazonaws.com. Leaving 0.0.0.0/0 exposes the login and dashboard pages to the whole internet. |
| :---- |

## **Step-by-step: apply the network stack**

cd kafka-keycloak-terraform  
terraform \-chdir=stacks/10-network init  
terraform \-chdir=stacks/10-network plan \-var-file=dev.tfvars  
terraform \-chdir=stacks/10-network apply \-var-file=dev.tfvars  
terraform \-chdir=stacks/10-network output

Expected plan size: about 16 resources (VPC, IGW, subnet, route table, route, association, 2 SGs, 6 rules, role, attachment, profile).

## **Verify with the AWS CLI**

cd stacks/10-network  
VPC=$(terraform output \-raw vpc\_id)  
SUB=$(terraform output \-json public\_subnet\_ids | jq \-r '."public-a"')  
aws ec2 describe-vpcs \--vpc-ids $VPC \--query 'Vpcs\[0\].{Cidr:CidrBlock,DnsHost:EnableDnsHostnames}'   
aws ec2 describe-subnets \--subnet-ids $SUB \--query 'Subnets\[0\].{AZ:AvailabilityZone,Cidr:CidrBlock,Public:MapPublicIpOnLaunch}'  
aws ec2 describe-route-tables \--filters Name=association.subnet-id,Values=$SUB \--query 'RouteTables\[0\].Routes'  
aws ec2 describe-instance-type-offerings \--location-type availability-zone \--filters Name=instance-type,Values=t3.medium \--query 'InstanceTypeOfferings\[\].Location'  
terraform state list

## **Chapter Quiz**

**Q1. Why sort the common AZ list?**

a) AWS requires it

b) Sets are unordered; sorting makes the choice deterministic

c) For speed

d) To pick the cheapest

**Q2. What does the ... in setintersection(local.az\_sets...) do?**

a) Comments out the rest

b) Spreads the list into separate arguments

c) Repeats the call

d) Nothing

**Q3. What replaces "ALLOWED\_CIDR" in tfvars?**

a) Nothing; it's literal

b) The value of var.allowed\_cidr, via a local

c) Your AWS account ID

d) The VPC CIDR

**Q4. What do default\_tags in the provider do?**

a) Tag only the VPC

b) Apply tags to every resource the provider creates

c) Replace Name tags

d) Nothing without a variable

**Q5. The outputs of the network stack are best described as:**

a) Debug info

b) The contract the apps stack depends on

c) Temporary

d) Sensitive

# **Chapter 19: The Apps Stack (Process 2), Line by Line**

Folder: stacks/20-apps/. This stack builds two servers on top of the network. It is the most detailed chapter, because this is where Terraform, shell scripts, Docker and the applications meet.

## **variables.tf – the important ones**

variable "network\_state" {  
  type \= object({ backend \= string, config \= map(string) })  
  default \= { backend \= "local", config \= { path \= "../10-network/terraform.tfstate" } }  
}  
   
variable "network" {  
  type \= object({  
    subnet\_key                  \= optional(string, "public-a")  
    keycloak\_security\_group\_key \= optional(string, "keycloak")  
    kafka\_security\_group\_key    \= optional(string, "kafka")  
    instance\_profile\_key        \= optional(string, "ec2\_ssm")  
    vpc\_id                      \= optional(string)  
    subnet\_id                   \= optional(string)  
    keycloak\_security\_group\_ids \= optional(list(string))  
    kafka\_security\_group\_ids    \= optional(list(string))  
    instance\_profile\_name       \= optional(string)  
  })  
  default \= {}  
}  
   
variable "ami\_id"            { type \= string, default \= null }  
variable "ami\_ssm\_parameter" { type \= string, default \= "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86\_64" }  
   
variable "keycloak" { type \= object({ enabled, instance\_type, root\_volume\_size, root\_volume\_type, key\_name, admin\_username, admin\_password, tags }) }  
variable "kafka"    { type \= object({ enabled, instance\_type, root\_volume\_size, root\_volume\_type, key\_name, ui\_username, ui\_password, tags }) }

| Variable | Why |
| :---- | :---- |
| network\_state | Where the network stack's state lives — local file by default, S3 for teams |
| network.\*\_key | Which subnet/SG/profile to use, *by name* — no IDs in tfvars |
| network.\*\_id / \_ids / \_name | Escape hatch: pass raw IDs and skip remote state (deploy into any existing VPC) |
| ami\_id / ami\_ssm\_parameter | Pin an AMI, or always take the latest Amazon Linux 2023 |
| keycloak, kafka objects | One variable per server holding everything about it |

## **data.tf – reading the network and the AMI**

data "terraform\_remote\_state" "network" {  
  count   \= local.needs\_remote\_state ? 1 : 0  
  backend \= var.network\_state.backend  
  config  \= var.network\_state.config  
}  
   
data "aws\_ssm\_parameter" "ami" {  
  count \= var.ami\_id \== null ? 1 : 0  
  name  \= var.ami\_ssm\_parameter  
}

| Line | What | Why / How |
| :---- | :---- | :---- |
| terraform\_remote\_state | Read another stack's outputs | Only its *outputs* are visible — not its resources |
| count \= needs\_remote\_state ? 1 : 0 | Skip the read if every ID was overridden | Lets the stack run without the network stack |
| aws\_ssm\_parameter | Public parameter with the newest AL2023 AMI ID | AWS updates it; you get patched images automatically |

| GOTCHA Because the AMI parameter changes over time, a plan months later will show the launch template changing and the instance being replaced. For production, pin ami\_id and update deliberately. |
| :---- |

## **locals.tf – resolving inputs**

locals {  
  needs\_remote\_state \= anytrue(\[  
    var.network.subnet\_id \== null,  
    var.network.keycloak\_security\_group\_ids \== null,  
    var.network.kafka\_security\_group\_ids \== null,  
    var.network.instance\_profile\_name \== null,  
  \])  
   
  net \= try(data.terraform\_remote\_state.network\[0\].outputs, {})  
   
  subnet\_id \= var.network.subnet\_id \!= null ? var.network.subnet\_id : try(local.net.public\_subnet\_ids\[var.network.subnet\_key\], null)  
  keycloak\_security\_group\_ids \= var.network.keycloak\_security\_group\_ids \!= null ? var.network.keycloak\_security\_group\_ids : try(\[local.net.security\_group\_ids\[var.network.keycloak\_security\_group\_key\]\], null)  
  kafka\_security\_group\_ids    \= ...same pattern...  
  instance\_profile\_name       \= ...same pattern...  
   
  ami\_id \= var.ami\_id \!= null ? var.ami\_id : try(data.aws\_ssm\_parameter.ami\[0\].value, null)  
   
  keycloak\_admin\_password \= var.keycloak.admin\_password \!= null ? var.keycloak.admin\_password : try(random\_password.keycloak\_admin\[0\].result, null)  
  kafka\_ui\_password       \= var.kafka.ui\_password \!= null ? var.kafka.ui\_password : try(random\_password.kafka\_ui\_user\[0\].result, null)  
  kafka\_ui\_client\_secret  \= random\_password.kafka\_ui\_client\_secret.result  
}

Every local follows the same shape: **"if the user gave me a value, use it; otherwise look it up."** This pattern — explicit override, then discovery — is what makes the stack reusable.

## **main.tf – secrets**

resource "random\_password" "keycloak\_admin" {  
  count   \= var.keycloak.admin\_password \== null ? 1 : 0  
  length  \= 24  
  special \= false  
}  
resource "random\_password" "kafka\_ui\_user"          { count \= var.kafka.ui\_password \== null ? 1 : 0, length \= 20, special \= false }  
resource "random\_password" "kafka\_ui\_client\_secret" { length \= 32, special \= false }

| Line | Why |
| :---- | :---- |
| random\_password | Generates a password once and stores it in state; it does not change on later applies |
| count \= ... \== null | Only generate when the user did not supply one |
| special \= false | Avoids characters that break shell scripts or YAML |
| kafka\_ui\_client\_secret | The OIDC "client secret" shared between Kafka UI and Keycloak — always generated |

| NOTE These values are in the state file. That is one more reason to protect state (Chapter 22 uses S3 with encryption). |
| :---- |

## **main.tf – Elastic IPs first**

module "keycloak\_eip" {  
  source \= "../../modules/elastic\_ip"  
  count  \= var.keycloak.enabled ? 1 : 0  
  name   \= "${var.project\_name}-keycloak"  
}  
module "kafka\_eip" { ...same for kafka... }

Allocated before anything else so the addresses can be baked into configuration files.

## **main.tf – rendering Keycloak's configuration**

locals {  
  keycloak\_public\_ip \= try(module.keycloak\_eip\[0\].public\_ip, null)  
  kafka\_public\_ip    \= try(module.kafka\_eip\[0\].public\_ip, "kafka-ui.invalid")  
   
  keycloak\_realm \= templatefile("${path.module}/files/keycloak-realm.json.tftpl", {  
    kafka\_public\_ip   \= local.kafka\_public\_ip  
    kafka\_ui\_username \= var.kafka.ui\_username  
    kafka\_ui\_password \= local.kafka\_ui\_password  
    kafka\_ui\_secret   \= local.kafka\_ui\_client\_secret  
  })  
   
  keycloak\_compose \= templatefile("${path.module}/files/keycloak-compose.yml.tftpl", {  
    keycloak\_public\_ip      \= local.keycloak\_public\_ip  
    keycloak\_admin\_username \= var.keycloak.admin\_username  
    keycloak\_admin\_password \= local.keycloak\_admin\_password  
  })  
   
  keycloak\_user\_data \= templatefile("${path.module}/templates/keycloak\_user\_data.sh.tftpl", {  
    keycloak\_compose\_b64 \= base64encode(local.keycloak\_compose)  
    keycloak\_realm\_b64   \= base64encode(local.keycloak\_realm)  
    keycloak\_public\_ip   \= local.keycloak\_public\_ip  
  })  
}

Three layers of templates, like nesting boxes:

1. **Realm JSON** – Keycloak's definition of the kafka-ui client (with the secret and the redirect URL http://\<kafka-ip\>:8080/login/oauth2/code/keycloak) and the demo user.

2. **docker-compose.yml** – how to run the Keycloak container: admin username/password, KC\_HOSTNAME=https://\<keycloak-ip\>:8443, HTTPS certificate paths, health and metrics on 9000\.

3. **user data script** – installs Docker, generates a self-signed TLS certificate whose SANs include the public IP, the private IP and 127.0.0.1, writes the two files above (decoded from base64), creates a systemd service, and waits until Keycloak answers.

| Why base64? | Compose YAML and realm JSON contain quotes, newlines and $ signs. Base64 turns them into one safe string that the shell script decodes with base64 \-d. |
| :---- | :---- |

| REAL-WORLD ANALOGY The realm is the school roster and door list; the compose file is the daily schedule; user data is the instructions to the janitor for opening day: "unlock the building, post the schedule, hang the roster." |
| :---- |

## **The Keycloak user-data script explained**

Key parts of templates/keycloak\_user\_data.sh.tftpl:

\#\!/bin/bash  
set \-euxo pipefail                    \# stop on any error, print each command  
exec \> \>(tee \-a /var/log/keycloak-bootstrap.log | logger \-t user-data \-s 2\>/dev/console) 2\>&1  
   
dnf install \-y docker openssl         \# Amazon Linux 2023 package manager  
systemctl enable \--now docker  
\# download the docker compose plugin (pinned version)  
   
IMDS\_TOKEN=$(curl \-X PUT \-H 'X-aws-ec2-metadata-token-ttl-seconds: 21600' http://169.254.169.254/latest/api/token)  
PRIVATE\_IP=$(curl \-H "X-aws-ec2-metadata-token: $IMDS\_TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4)  
PUBLIC\_IP="${keycloak\_public\_ip}"    \# \<- filled in by Terraform  
   
\# openssl.cnf with subjectAltName \= IP.1=$PUBLIC\_IP, IP.2=$PRIVATE\_IP, IP.3=127.0.0.1  
openssl req \-x509 \-nodes \-newkey rsa:2048 \-days 3650 \-keyout keycloak.key \-out keycloak.crt \-config openssl.cnf  
   
echo '${keycloak\_compose\_b64}' | base64 \-d \> docker-compose.yml  
echo '${keycloak\_realm\_b64}'   | base64 \-d \> keycloak-realm.json  
chown 1000:0 keycloak-realm.json      \# the Keycloak container runs as UID 1000  
   
\# systemd unit keycloak-compose.service \-\> docker compose up \-d  
systemctl enable \--now keycloak-compose.service  
   
for i in {1..60}; do curl \--cacert keycloak.crt \-fsS https://127.0.0.1:8443/realms/kafka-ui/.well-known/openid-configuration && break; sleep 2; done

| Step | Why |
| :---- | :---- |
| set \-euxo pipefail | Fail fast and log every command — you can read the log later via SSM |
| exec \> \>(tee ...) | Send all output to a log file *and* the console (visible with get-console-output) |
| IMDSv2 token dance | Because we set http\_tokens \= required, the script must fetch a token before reading metadata |
| Self-signed cert with three SANs | Browsers reach the public IP, Kafka UI reaches the private IP, health checks use localhost — all must be in the certificate or TLS fails |
| chown 1000:0 | The container's user must be able to read the files; a classic Docker permissions gotcha |
| systemd service | Survives reboots; docker compose up \-d runs again automatically |
| Readiness loop | User data reports success only after Keycloak actually answers |

## **main.tf – the Keycloak server**

module "keycloak\_launch\_template" {  
  source                    \= "../../modules/launch\_template"  
  count                     \= var.keycloak.enabled ? 1 : 0  
  name                      \= "${var.project\_name}-keycloak"  
  image\_id                  \= local.ami\_id  
  instance\_type             \= var.keycloak.instance\_type  
  key\_name                  \= var.keycloak.key\_name  
  iam\_instance\_profile\_name \= local.instance\_profile\_name  
  subnet\_id                 \= local.subnet\_id  
  security\_group\_ids        \= local.keycloak\_security\_group\_ids  
  user\_data                 \= local.keycloak\_user\_data  
  root\_volume\_size          \= var.keycloak.root\_volume\_size  
  root\_volume\_type          \= var.keycloak.root\_volume\_type  
  tags                      \= merge(var.tags, var.keycloak.tags)  
}  
   
module "keycloak" {  
  source                  \= "../../modules/ec2\_instance"  
  count                   \= var.keycloak.enabled ? 1 : 0  
  name                    \= "${var.project\_name}-keycloak"  
  launch\_template\_id      \= module.keycloak\_launch\_template\[0\].id  
  launch\_template\_version \= tostring(module.keycloak\_launch\_template\[0\].latest\_version)  
  eip\_allocation\_id       \= module.keycloak\_eip\[0\].allocation\_id  
  tags                    \= merge(var.tags, var.keycloak.tags)  
}

Every input comes from a var. or a local. — the file is pure wiring.

## **main.tf – Kafka's configuration depends on Keycloak**

locals {  
  keycloak\_private\_ip \= try(module.keycloak\[0\].private\_ip, null)  
   
  kafka\_ui\_config \= templatefile("${path.module}/files/kafka-ui.yml.tftpl", {  
    kafka\_public\_ip     \= local.kafka\_public\_ip  
    keycloak\_public\_ip  \= local.keycloak\_public\_ip  
    keycloak\_private\_ip \= local.keycloak\_private\_ip  
    kafka\_ui\_secret     \= local.kafka\_ui\_client\_secret  
  })  
  kafka\_compose   \= templatefile("${path.module}/files/docker-compose.yml.tftpl", {})  
  kafka\_user\_data \= templatefile("${path.module}/templates/user\_data.sh.tftpl", {  
    docker\_compose\_b64  \= base64encode(local.kafka\_compose)  
    kafka\_ui\_config\_b64 \= base64encode(local.kafka\_ui\_config)  
    keycloak\_private\_ip \= local.keycloak\_private\_ip  
  })  
}

keycloak\_private\_ip is only known **after** the Keycloak instance exists. Because the Kafka user data references it, Terraform automatically builds Keycloak first. This is an implicit dependency doing real work.

The kafka-ui.yml template is worth reading:

auth:  
  type: OAUTH2  
  oauth2:  
    client:  
      keycloak:  
        clientId: kafka-ui  
        clientSecret: "${kafka\_ui\_secret}"  
        redirect-uri: "http://${kafka\_public\_ip}:8080/login/oauth2/code/keycloak"  
        authorization-uri: "https://${keycloak\_public\_ip}:8443/realms/kafka-ui/protocol/openid-connect/auth"  
        token-uri:     "https://${keycloak\_private\_ip}:8443/realms/kafka-ui/protocol/openid-connect/token"  
        jwk-set-uri:   "https://${keycloak\_private\_ip}:8443/realms/kafka-ui/protocol/openid-connect/certs"  
        user-info-uri: "https://${keycloak\_private\_ip}:8443/realms/kafka-ui/protocol/openid-connect/userinfo"

| URL | Who calls it | Address used |
| :---- | :---- | :---- |
| authorization-uri | Your **browser** (redirected by Kafka UI) | Keycloak **public** IP |
| token-uri, jwk-set-uri, user-info-uri | The **Kafka UI server** | Keycloak **private** IP, inside the VPC |

This split is why the Keycloak security group has two rules on 8443: one for your IP (browser), one for the Kafka SG (server-to-server).

## **The Kafka user-data script explained**

templates/user\_data.sh.tftpl does the same Docker setup, then:

KEYCLOAK\_PRIVATE\_IP="${keycloak\_private\_ip}"  
for i in {1..90}; do  
  openssl s\_client \-connect "$KEYCLOAK\_PRIVATE\_IP:8443" \-showcerts \</dev/null | openssl x509 \-outform PEM \> keycloak.crt && test \-s keycloak.crt && break  
  sleep 2  
done  
docker run \--rm \--entrypoint keytool ghcr.io/kafbat/kafka-ui:v1.5.0 \-importcert \-noprompt \-alias keycloak \-file /work/keycloak.crt \-keystore /work/keycloak-truststore.p12 \-storetype PKCS12 \-storepass changeit  
curl \--cacert keycloak.crt \-fsS https://$KEYCLOAK\_PRIVATE\_IP:8443/realms/kafka-ui/.well-known/openid-configuration  
systemctl enable \--now kafka-compose.service

| Step | Why |
| :---- | :---- |
| Loop fetching Keycloak's certificate | Keycloak may still be starting; retry up to 3 minutes |
| keytool \-importcert into a PKCS12 truststore | Kafka UI is a Java app; Java only trusts certificates in its truststore. We use the *same image's* keytool so the format is exactly right |
| curl \--cacert check | Prove TLS works before starting Kafka UI, so failures are obvious in the log |

The docker-compose.yml for Kafka runs apache/kafka:4.3.1 in KRaft mode (single node acting as both broker and controller) and kafbat/kafka-ui:v1.5.0 with JAVA\_OPTS pointing at the truststore.

## **main.tf – the Kafka server and the explicit dependency**

module "kafka" {  
  source                  \= "../../modules/ec2\_instance"  
  count                   \= var.kafka.enabled ? 1 : 0  
  launch\_template\_id      \= module.kafka\_launch\_template\[0\].id  
  launch\_template\_version \= tostring(module.kafka\_launch\_template\[0\].latest\_version)  
  eip\_allocation\_id       \= module.kafka\_eip\[0\].allocation\_id  
  depends\_on              \= \[module.keycloak\]  
}  
   
check "kafka\_requires\_keycloak" {  
  assert {  
    condition     \= \!var.kafka.enabled || var.keycloak.enabled  
    error\_message \= "Kafka UI authenticates against Keycloak; enable keycloak when kafka is enabled."  
  }  
}

The launch template already depends on Keycloak's private IP, so why depends\_on? Because Terraform only guarantees Keycloak *exists* before the template is created — it might still be booting. depends\_on \= \[module.keycloak\] also waits for the EIP association inside that module, giving Keycloak's user data a head start before Kafka's script begins polling.

## **outputs.tf**

output "kafka\_ui\_url"           { value \= try("http://${module.kafka\_eip\[0\].public\_ip}:8080", null) }  
output "keycloak\_admin\_url"     { value \= local.keycloak\_public\_ip \== null ? null : "https://${local.keycloak\_public\_ip}:8443/admin/" }  
output "keycloak\_admin\_password" { value \= local.keycloak\_admin\_password, sensitive \= true }  
output "kafka\_ui\_user\_password"  { value \= local.kafka\_ui\_password,       sensitive \= true }  
output "ssm\_keycloak" { value \= try("aws ssm start-session \--region ${var.aws\_region} \--target ${module.keycloak\[0\].id}", null) }  
output "network\_inputs\_resolved" { value \= { subnet\_id \= local.subnet\_id, ... } }

network\_inputs\_resolved is a debugging aid: it shows exactly which subnet, SGs, profile and AMI the stack decided to use.

## **dev.tfvars**

network\_state \= { backend \= "local", config \= { path \= "../10-network/terraform.tfstate" } }  
network \= {  
  subnet\_key \= "public-a", keycloak\_security\_group\_key \= "keycloak",  
  kafka\_security\_group\_key \= "kafka", instance\_profile\_key \= "ec2\_ssm"  
}  
ami\_id \= null  
keycloak \= { enabled \= true, instance\_type \= "t3.small",  root\_volume\_size \= 20, admin\_username \= "admin" }  
kafka    \= { enabled \= true, instance\_type \= "t3.medium", root\_volume\_size \= 30, ui\_username \= "kafkauser" }

## **Chapter Quiz**

**Q1. What can terraform\_remote\_state read from another stack?**

a) All resources

b) Only its outputs

c) Its variables

d) Its tfvars

**Q2. Why is Keycloak built before Kafka even without depends\_on?**

a) Alphabetical order

b) Kafka's user data references Keycloak's private IP

c) Keycloak is smaller

d) It isn't

**Q3. Why does Kafka UI use the private IP for token-uri but the public IP for authorization-uri?**

a) A typo

b) The browser needs a public address; the server-to-server call stays inside the VPC

c) Private is faster

d) Keycloak requires it

**Q4. Why base64-encode compose files into user data?**

a) Encryption

b) To safely carry quotes, newlines and $ through a shell script

c) AWS limits

d) Docker requires it

**Q5. Where does a generated random\_password live?**

a) In a tag

b) In the state file

c) On the instance only

d) In tfvars

# **Chapter 20: Deploy, Test and Tear Down — Step by Step**

## **Before you start**

1. aws sts get-caller-identity works.

2. Terraform ≥ 1.6 installed.

3. curl \-s https://checkip.amazonaws.com → put this in stacks/10-network/dev.tfvars as allowed\_cidr \= "X.X.X.X/32".

4. jq installed (optional, for parsing JSON outputs).

## **Step 1 – Network**

make network ENV=dev  
\# or  
terraform \-chdir=stacks/10-network init  
terraform \-chdir=stacks/10-network apply \-var-file=dev.tfvars

Read the plan. You should see only \+ (create) lines. Type yes. Takes about 30 seconds.

## **Step 2 – Apps**

make apps ENV=dev

Expect: 3 random passwords, 2 EIPs, 2 launch templates, 2 instances, 2 EIP associations. Terraform finishes in \~2 minutes, but the servers keep working for another 3–6 minutes running user data.

## **Step 3 – Watch the boot**

cd stacks/20-apps  
terraform output  
KC=$(terraform output \-raw keycloak\_instance\_id)  
KF=$(terraform output \-raw kafka\_instance\_id)  
   
\# Console output (may lag a few minutes)  
aws ec2 get-console-output \--instance-id $KC \--latest \--output text | tail \-30  
   
\# Or open a shell and read the bootstrap log  
aws ssm start-session \--target $KC  
  sudo tail \-f /var/log/keycloak-bootstrap.log  
  sudo docker ps  
  exit

| TIP If start-session says the target is not connected, wait a minute — the SSM agent registers after boot — and confirm with aws ssm describe-instance-information. |
| :---- |

## **Step 4 – Test Keycloak**

1. Open terraform output \-raw keycloak\_admin\_url in a browser.

2. Accept the self-signed certificate warning (expected in a lab).

3. Log in with admin and terraform output \-raw keycloak\_admin\_password.

4. Switch realm to **kafka-ui**; confirm the client kafka-ui and user kafkauser exist.

KC\_IP=$(terraform output \-raw keycloak\_public\_ip)  
curl \-k https://$KC\_IP:9000/health/ready  
curl \-k https://$KC\_IP:8443/realms/kafka-ui/.well-known/openid-configuration | jq .issuer

## **Step 5 – Test Kafka UI login**

1. Open terraform output \-raw kafka\_ui\_url.

2. You are redirected to Keycloak. Log in as kafkauser with terraform output \-raw kafka\_ui\_user\_password.

3. You land in Kafka UI showing cluster local-kafka. Create a topic and send a message to prove Kafka works.

## **Step 6 – Verify everything with the CLI**

aws ec2 describe-instances \--filters Name=tag:Project,Values=kafka-keycloak-lab \\  
  \--query 'Reservations\[\].Instances\[\].{Name:Tags\[?Key==\`Name\`\].Value|\[0\],State:State.Name,Type:InstanceType,Public:PublicIpAddress,Private:PrivateIpAddress}' \--output table  
aws ec2 describe-addresses \--output table  
aws ssm describe-instance-information \--query 'InstanceInformationList\[\].{Id:InstanceId,Ping:PingStatus,Agent:AgentVersion}'  
aws ec2 describe-instance-status \--include-all-instances \--query 'InstanceStatuses\[\].{Id:InstanceId,Sys:SystemStatus.Status,Inst:InstanceStatus.Status}'

## **Step 7 – Make a change and re-apply**

Change kafka.instance\_type to t3.large in dev.tfvars, then terraform plan \-var-file=dev.tfvars. Notice: the launch template updates in place (new version), and the instance shows \-/+ (replace) because the version changed. That is Terraform being honest about what a resize requires.

## **Step 8 – Tear down**

make destroy ENV=dev  
\# \= apps destroy, then network destroy

Order matters: instances must go before the subnet and security groups they use. Terraform handles the order inside a stack, but across stacks *you* run them in reverse.

| GOTCHA If you destroy the network stack first, Terraform will fail with "DependencyViolation" because instances still use the subnet. Always apps first. |
| :---- |

## **Troubleshooting table**

| Symptom | Likely cause | Check |
| :---- | :---- | :---- |
| Kafka UI shows TLS error to Keycloak | Kafka booted before Keycloak was ready | /var/log/kafka-bootstrap.log on Kafka host; re-run systemctl restart kafka-compose after Keycloak is up |
| Browser cannot reach 8080/8443 | allowed\_cidr is not your IP | curl checkip.amazonaws.com, update tfvars, apply network stack |
| SSM session fails | Agent not registered / no internet route | describe-instance-information; confirm subnet is public |
| "No AZ supports both instance types" | Region lacks those types together | Set availability\_zone or change types |
| Plan wants to replace instances unexpectedly | AMI SSM parameter updated | Pin ami\_id |
| DependencyViolation on destroy | Wrong stack order | Destroy apps first |

## **Chapter Quiz**

**Q1. In what order do you destroy the stacks?**

a) Network then apps

b) Apps then network

c) Either

d) Both at once

**Q2. How do you get a shell on an instance in this project?**

a) SSH with a key on port 22

b) aws ssm start-session

c) The AWS console only

d) telnet

**Q3. Where is the Keycloak bootstrap log?**

a) /var/log/messages

b) /var/log/keycloak-bootstrap.log

c) /opt/keycloak/log

d) CloudWatch by default

**Q4. Changing an instance type in tfvars causes:**

a) In-place resize

b) A new launch template version and instance replacement

c) Nothing

d) An error

**Q5. Which command proves Keycloak is ready?**

a) curl the health/ready endpoint on 9000

b) ping

c) terraform apply

d) docker version

# **Chapter 21: Best Practices, Tips and Gotchas**

This chapter gathers the professional habits that separate a lab from a system you can trust. Each item says what to do, why, and where it shows up in the project.

## **Code organisation**

| BEST PRACTICE One resource per concern, one module per building block, one stack per blast radius. Our split (modules → network stack → apps stack) means a mistake in the apps stack cannot delete the VPC. |
| :---- |

| BEST PRACTICE Standard file names. main.tf, variables.tf, outputs.tf, versions.tf, providers.tf, locals.tf, data.tf. Newcomers know where to look. |
| :---- |

| BEST PRACTICE No values in modules. Every module input is a variable. Values live in tfvars. If you find yourself typing "10.40.0.0/16" inside a module, stop. |
| :---- |

| BEST PRACTICE Run \`terraform fmt\` before every commit and terraform validate in CI. Formatting fights are wasted review time. |
| :---- |

## **Naming and tagging**

| BEST PRACTICE Use name\_prefix for resources that may need replacing (SGs, launch templates, roles) so create-before-destroy works. |
| :---- |

| BEST PRACTICE Use default\_tags on the provider for Project / Environment / Owner / ManagedBy. Tag volumes via launch template tag\_specifications — they are easy to forget and show up as mystery costs. |
| :---- |

## **Variables and tfvars**

| TIP Prefer object({ ... optional(...) }) variables for "one thing with many settings" (like our keycloak object) and plain variables for cross-cutting values (aws\_region). |
| :---- |

| TIP Give every variable a description. Add validation blocks for formats (CIDRs, instance types) so mistakes fail in seconds, not after AWS rejects them. |
| :---- |

| GOTCHA terraform.tfvars is auto-loaded; dev.tfvars is not. Always pass \-var-file= explicitly or you will apply defaults by accident. |
| :---- |

## **State**

| BEST PRACTICE Remote state (S3 \+ native locking or DynamoDB) for anything with more than one person or one machine. Enable versioning on the bucket so you can recover. |
| :---- |

| GOTCHA Never run two applies at once against the same state. Locking exists to stop this; do not disable it. |
| :---- |

| TIP terraform state list, terraform state show, terraform state mv and terraform import are your tools for fixing state. Back it up before using mv. |
| :---- |

## **Security**

| BEST PRACTICE http\_tokens \= "required" (IMDSv2), encrypted volumes, no SSH ports, SSM Session Manager — all already in the project. |
| :---- |

| BEST PRACTICE Least-privilege IAM. The lab role has only AmazonSSMManagedInstanceCore. Add permissions per need, never AdministratorAccess on instances. |
| :---- |

| BEST PRACTICE allowed\_cidr \= YOUR\_IP/32. Never leave admin consoles on 0.0.0.0/0. |
| :---- |

| BEST PRACTICE Mark secrets sensitive \= true; keep them out of tfvars in Git. Prefer AWS Secrets Manager or SSM SecureString for real systems (Chapter 22). |
| :---- |

## **Plans and applies**

| BEST PRACTICE Read every plan. Search for "forces replacement" and "destroy". Save plans in CI (terraform plan \-out=tfplan) and apply exactly that file. |
| :---- |

| TIP terraform plan \-target=module.vpc limits a plan to one thing while debugging. Do not use \-target for normal applies; it leaves state incomplete. |
| :---- |

| GOTCHA Data sources refresh every plan. A data source that returns a different value (latest AMI) causes drift. Pin values in production. |
| :---- |

## **Modules and versions**

| BEST PRACTICE Pin provider versions with \~\>. Commit .terraform.lock.hcl. Pin remote module versions. |
| :---- |

| GOTCHA Upgrading a major provider version (5 → 6\) can rename attributes. Read the upgrade guide, run plan in a sandbox first. |
| :---- |

## **User data and boot scripts**

| TIP set \-euxo pipefail and log to a file. Poll for readiness instead of sleep 60. |
| :---- |

| GOTCHA User data has a 16 KB limit. Our scripts embed base64 compose files and are close. For bigger payloads, put files in S3 and download them with the instance role. |
| :---- |

| GOTCHA Changing user data does not touch a running instance. Plan for replacement. |
| :---- |

## **Cost**

| TIP A running lab (t3.small \+ t3.medium \+ 2 EIPs) costs roughly $2/day. Destroy when done. Unattached EIPs and forgotten NAT Gateways are the classic surprise bills. |
| :---- |

| TIP aws ce get-cost-and-usage or the Cost Explorer console, filtered by the Project tag, shows exactly what this project costs. |
| :---- |

## **Quick command reference**

| Task | Command |
| :---- | :---- |
| Format | terraform fmt \-recursive |
| Validate | terraform validate |
| Plan to file | terraform plan \-var-file=dev.tfvars \-out=tfplan |
| Apply saved plan | terraform apply tfplan |
| Show outputs | terraform output / terraform output \-raw name / \-json |
| List state | terraform state list |
| Inspect one | terraform state show module.vpc.aws\_vpc.this |
| Force replace | terraform apply \-replace=module.kafka\[0\].aws\_instance.this |
| Import existing | terraform import module.vpc.aws\_vpc.this vpc-0abc... |
| Console (try expressions) | terraform console |
| Dependency graph | terraform graph, then pipe to Graphviz dot \-Tpng |

## **Chapter Quiz**

**Q1. Which file is auto-loaded by Terraform?**

a) dev.tfvars

b) terraform.tfvars

c) variables.tf only

d) any .tfvars

**Q2. Why save a plan to a file in CI?**

a) Faster

b) So the apply executes exactly what was reviewed

c) Required by AWS

d) For logging only

**Q3. What is the risk of \-target in normal applies?**

a) It is slow

b) It leaves state and dependencies partially applied

c) It deletes resources

d) Nothing

**Q4. Best place for real secrets?**

a) tfvars in Git

b) Hard-coded in main.tf

c) Secrets Manager / SSM SecureString, referenced at runtime

d) A tag

**Q5. What is the user-data size limit?**

a) 1 KB

b) 16 KB

c) 1 MB

d) Unlimited

# **Chapter 22: Making the Project Production-Ready**

The lab works, but a real company would need more. This chapter walks through each upgrade, why it matters, and sketches the Terraform for it. Think of it as the renovation plan that turns a temporary classroom trailer into a permanent school.

## **1\. Remote state with locking**

**Why:** Teams and CI need shared, locked, versioned state.

\# stacks/10-network/versions.tf  
terraform {  
  backend "s3" {  
    bucket       \= "acme-tfstate-prod"  
    key          \= "kafka-keycloak/network.tfstate"  
    region       \= "us-east-1"  
    encrypt      \= true  
    use\_lockfile \= true          \# native S3 locking (Terraform 1.10+); or dynamodb\_table \= "tf-locks"  
  }  
}

And in stacks/20-apps/prod.tfvars:

network\_state \= { backend \= "s3", config \= { bucket \= "acme-tfstate-prod", key \= "kafka-keycloak/network.tfstate", region \= "us-east-1" } }

Create the bucket once (by hand or a tiny "bootstrap" stack) with versioning and SSE-KMS enabled, and a bucket policy limited to the CI role.

## **2\. Private subnets, load balancers and real certificates**

**Why:** Servers should not have public IPs. Users should hit HTTPS with a certificate their browser trusts.

* Add private\_subnets in two AZs and create\_nat\_gateway \= true (the VPC module already supports this).

* Put Keycloak and Kafka UI behind an **Application Load Balancer** in the public subnets with an **ACM** certificate for a real domain (auth.example.com, kafka-ui.example.com) and a **Route 53** record.

* Change SG rules: ALB SG allows 443 from the internet (or your office); instance SGs allow 8443/8080 **only from the ALB SG** (source\_security\_group\_key \= "alb").

* Remove Elastic IPs; set associate\_public\_ip\_address \= false.

* Set KC\_HOSTNAME=https://auth.example.com and Kafka UI redirect/issuer URLs to the domain; the self-signed certificate and truststore steps disappear.

Suggested new module: modules/alb (ALB, listener, target group, health check). Suggested tfvars shape:

keycloak \= { ..., hostname \= "auth.example.com", certificate\_arn \= "arn:aws:acm:..." }

## **3\. Managed database for Keycloak**

**Why:** Keycloak's H2 file database is for development only. Losing the disk loses every user.

* Add modules/rds creating **Amazon RDS for PostgreSQL** (Multi-AZ, encrypted, automated backups) in the private subnets.

* Keycloak env: KC\_DB=postgres, KC\_DB\_URL, KC\_DB\_USERNAME, KC\_DB\_PASSWORD (from Secrets Manager).

* Run Keycloak with start (production mode) instead of start-dev.

## **4\. Secrets management**

**Why:** Passwords should not live in state or user data.

* Store admin password, DB password and OIDC client secret in **AWS Secrets Manager** (aws\_secretsmanager\_secret \+ \_version).

* Give the instance role secretsmanager:GetSecretValue on those ARNs only (an inline\_policies entry on the ec2\_ssm role — the IAM module already supports it).

* User data fetches secrets at boot with aws secretsmanager get-secret-value instead of embedding them. User data shrinks and no longer needs to be sensitive.

## **5\. Resilience: Auto Scaling Groups and health checks**

**Why:** A single instance that dies stays dead.

* Replace aws\_instance with an **Auto Scaling Group** (min 1, max 1, or more for Kafka UI) using the same launch template. The launch\_template module needs no change — that is the payoff of the modular design.

* ALB target-group health checks (/health/ready for Keycloak, /actuator/health for Kafka UI) let the ASG replace unhealthy instances.

* For Kafka itself, move to **Amazon MSK** (managed Kafka, multi-broker, multi-AZ) or run 3 brokers on separate instances with EBS data volumes (additional\_volumes in the launch template module).

## **6\. Observability**

**Why:** You cannot fix what you cannot see.

* Install the **CloudWatch agent** in user data; ship /var/log/\*-bootstrap.log and Docker logs to CloudWatch Logs.

* Enable detailed\_monitoring \= true in the launch template module.

* Create **CloudWatch alarms** (CPU, status checks, ALB 5xx, target unhealthy) → **SNS** → email/Slack.

* Scrape Keycloak's /metrics (already enabled on 9000\) with Prometheus or CloudWatch.

## **7\. Security hardening**

* **KMS customer-managed key** for EBS (root\_volume\_kms\_key\_id), RDS and Secrets Manager.

* **VPC Flow Logs** to S3/CloudWatch.

* **AWS Config** rules and **Security Hub** for continuous checks (e.g. "no SG open to 0.0.0.0/0 on admin ports").

* **Systems Manager Patch Manager** for OS updates; or bake a hardened AMI with **EC2 Image Builder** / Packer and pin ami\_id.

* termination\_protection \= true on production instances.

* IAM: replace the CI user's AdministratorAccess with a scoped role assumed via OIDC from GitHub Actions / GitLab.

## **8\. Backups and disaster recovery**

* **AWS Backup** plan for EBS volumes and RDS with a retention policy.

* Kafka: MSK handles replication; for self-managed, snapshot data volumes and set topic replication factor 3\.

* Document an RTO/RPO and test a restore twice a year.

## **9\. Multi-environment layout**

stacks/10-network/  
  dev.tfvars   staging.tfvars   prod.tfvars  
stacks/20-apps/  
  dev.tfvars   staging.tfvars   prod.tfvars

Same code, different values, separate state keys per environment. Prod tfvars: bigger instances, private subnets, allowed\_cidr \= office range, termination\_protection \= true.

## **10\. CI/CD pipeline**

A typical GitHub Actions flow:

1. On pull request: fmt \-check, validate, tflint, checkov, terraform plan for each stack → plan posted as a PR comment.

2. On merge to main: plan \-out, manual approval gate, apply network then apps.

3. Nightly: plan \-detailed-exitcode to detect drift; alert if exit code 2\.

## **Production checklist**

| Area | Lab | Production |
| :---- | :---- | :---- |
| State | Local file | S3 \+ lock \+ versioning \+ KMS |
| Network | 1 public subnet | 2+ AZs, private subnets, NAT, ALB |
| TLS | Self-signed | ACM certificate, real domain |
| Keycloak DB | H2 file | RDS PostgreSQL Multi-AZ |
| Kafka | Single node | MSK or 3+ brokers |
| Secrets | random\_password in state | Secrets Manager |
| Compute | aws\_instance | ASG \+ health checks |
| Access | allowed\_cidr | ALB \+ WAF, SSO for admin |
| Logs/metrics | On-box files | CloudWatch Logs, alarms |
| AMI | Latest via SSM | Pinned, patched by Image Builder |
| Deploy | Laptop | CI pipeline with approvals |

## **Chapter Quiz**

**Q1. Why move Keycloak off the H2 database?**

a) H2 is slow

b) H2 is a dev-only file DB; losing the disk loses all users

c) Terraform cannot manage H2

d) Licensing

**Q2. In the ALB design, who may reach port 8443 on the Keycloak instance?**

a) The internet

b) Only the ALB security group

c) Only your laptop

d) Nobody

**Q3. Which change requires NO edit to the launch\_template module?**

a) Switching from aws\_instance to an Auto Scaling Group

b) Changing to Azure

c) Removing Docker

d) Using Windows

**Q4. Where should the OIDC client secret live in production?**

a) tfvars

b) Secrets Manager, fetched at boot

c) A tag

d) In the AMI

**Q5. What does plan \-detailed-exitcode returning 2 mean?**

a) Error

b) No changes

c) Changes present (drift or pending change)

d) Locked

# **Chapter 23: Testing Terraform Projects**

Testing infrastructure code has layers, just like a building inspection: check the drawings, check the materials, then walk the finished building.

## **Layer 1 – Static checks (seconds, no AWS)**

| Tool | What it catches | Command |
| :---- | :---- | :---- |
| terraform fmt \-check | Formatting | terraform fmt \-check \-recursive |
| terraform validate | Syntax, types, missing args | per stack, after init \-backend=false |
| TFLint | Provider-specific mistakes (invalid instance type, deprecated args) | tflint \--recursive |
| Checkov / tfsec / Trivy | Security misconfigurations (open SGs, unencrypted disks, IMDSv1) | checkov \-d . |
| Infracost | Cost of a plan | infracost breakdown \--path . |

| TIP Our project already passes several Checkov checks by design: IMDSv2 required, EBS encrypted, no inline SG rules on 0.0.0.0/0 except what tfvars asks for. |
| :---- |

## **Layer 2 – Plan tests (seconds to minutes, no real resources)**

**Terraform's built-in test framework** (terraform test, 1.6+) runs .tftest.hcl files. With command \= plan, nothing is created.

stacks/10-network/tests/network.tftest.hcl:

variables {  
  project\_name      \= "test"  
  availability\_zone \= "us-east-1a"  
  allowed\_cidr      \= "203.0.113.10/32"  
  security\_groups \= {  
    web \= { description \= "web", ingress \= \[{ from\_port \= 443, to\_port \= 443, cidr\_ipv4 \= "ALLOWED\_CIDR" }\] }  
  }  
}  
   
run "allowed\_cidr\_is\_substituted" {  
  command \= plan  
  assert {  
    condition     \= local.security\_groups.web.ingress\[0\].cidr\_ipv4 \== "203.0.113.10/32"  
    error\_message \= "ALLOWED\_CIDR placeholder was not replaced"  
  }  
}  
   
run "one\_public\_subnet\_by\_default" {  
  command \= plan  
  assert {  
    condition     \= length(module.vpc.public\_subnet\_ids) \== 1  
    error\_message \= "expected exactly one public subnet"  
  }  
}

modules/security\_groups/tests/rules.tftest.hcl:

run "rejects\_rule\_with\_two\_sources" {  
  command \= plan  
  variables {  
    name   \= "t"  
    vpc\_id \= "vpc-123"  
    security\_groups \= { a \= { description \= "a", ingress \= \[{ from\_port \= 80, to\_port \= 80, cidr\_ipv4 \= "0.0.0.0/0", self \= true }\] } }  
  }  
  expect\_failures \= \[var.security\_groups\]  
}

Run with terraform test in the stack or module folder. Plan-mode tests still need provider credentials for some data sources; use mock\_provider blocks (Terraform 1.7+) to avoid that:

mock\_provider "aws" {}

## **Layer 3 – Integration tests (minutes, real resources, real money)**

Create, verify, destroy, in a sandbox account:

* **\`terraform test\` with \`command \= apply\`** – asserts against real outputs, then auto-destroys.

* **Terratest** (Go) – terraform.InitAndApply, then hit kafka\_ui\_url with HTTP and expect a redirect to Keycloak; defer terraform.Destroy.

* **Smoke script** – after apply, a bash script runs the Chapter 20 checks: SSM ping, /health/ready, OIDC discovery document, Kafka UI redirect.

Example smoke test:

\#\!/bin/bash  
set \-e  
cd stacks/20-apps  
KC=$(terraform output \-raw keycloak\_public\_ip)  
KU=$(terraform output \-raw kafka\_ui\_url)  
for i in {1..60}; do curl \-ksf https://$KC:9000/health/ready && break; sleep 10; done  
curl \-ksf https://$KC:8443/realms/kafka-ui/.well-known/openid-configuration | jq \-e '.issuer'   
code=$(curl \-s \-o /dev/null \-w '%{http\_code}' $KU)  
\[ "$code" \= "302" \] || \[ "$code" \= "200" \]  
echo "SMOKE TEST PASSED"

## **Layer 4 – Policy as code**

**OPA/Conftest** or **Sentinel** evaluate the plan JSON against rules like "no SG ingress from 0.0.0.0/0 on port 22" or "all instances must have IMDSv2":

terraform plan \-out=tfplan && terraform show \-json tfplan \> plan.json  
conftest test plan.json

## **Layer 5 – Drift detection and review**

* Nightly terraform plan \-detailed-exitcode per stack; alert on exit code 2\.

* Require two reviewers on plans that contain destroy or forces replacement.

* Keep terraform.tfstate versions in S3 so a bad apply can be compared and rolled back.

## **Test pyramid for this project**

| Level | How many | Speed | Where |
| :---- | :---- | :---- | :---- |
| fmt / validate / tflint / checkov | Every commit | Seconds | Local \+ CI |
| terraform test plan-mode | Dozens | Seconds | CI on PR |
| terraform test apply-mode / Terratest | A few | 10–15 min | CI nightly or on main |
| Smoke test after deploy | 1 script | 5 min | After every apply |
| Manual login walkthrough | 1 | 5 min | Before release |

## **Chapter Quiz**

**Q1. Which test layer creates no real resources?**

a) Terratest apply

b) terraform test with command \= plan

c) Smoke test

d) Manual login

**Q2. What does expect\_failures \= \[var.security\_groups\] assert?**

a) The plan succeeds

b) The variable's validation is expected to fail

c) AWS rejects it

d) Nothing

**Q3. What does Checkov look for?**

a) Formatting

b) Security misconfigurations

c) Cost

d) Typos in tags

**Q4. How does a smoke test know Kafka UI is protected by Keycloak?**

a) Ping

b) An HTTP 302 redirect to the Keycloak login page

c) Reading the state file

d) docker ps

**Q5. What is drift?**

a) A slow plan

b) Reality differing from state/code, e.g. someone changed a SG in the console

c) A failed test

d) A provider upgrade

# **Appendix A: Official Documentation and Further Reading**

## **Terraform**

* Terraform docs home: https://developer.hashicorp.com/terraform/docs

* Language reference: https://developer.hashicorp.com/terraform/language

* Functions: https://developer.hashicorp.com/terraform/language/functions

* Modules: https://developer.hashicorp.com/terraform/language/modules

* State: https://developer.hashicorp.com/terraform/language/state

* Backends (S3): https://developer.hashicorp.com/terraform/language/backend/s3

* terraform\_remote\_state: https://developer.hashicorp.com/terraform/language/state/remote-state-data

* Tests: https://developer.hashicorp.com/terraform/language/tests

* Style guide: https://developer.hashicorp.com/terraform/language/style

* CLI commands: https://developer.hashicorp.com/terraform/cli/commands

* AWS provider: https://registry.terraform.io/providers/hashicorp/aws/latest/docs

* Random provider: https://registry.terraform.io/providers/hashicorp/random/latest/docs

## **AWS**

* VPC: https://docs.aws.amazon.com/vpc/latest/userguide/

* Subnets and routing: https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html

* Security groups: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

* EC2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/

* Launch templates: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html

* Instance metadata (IMDSv2): https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html

* User data: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html

* Elastic IPs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html

* IAM roles for EC2: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html

* IAM best practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

* Session Manager: https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html

* Public AMI SSM parameters: https://docs.aws.amazon.com/linux/al2023/ug/ec2.html

* AWS CLI reference: https://docs.aws.amazon.com/cli/latest/

* JMESPath (for \--query): https://jmespath.org/tutorial.html

* Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/framework/

## **Applications**

* Keycloak server guide: https://www.keycloak.org/guides

* Keycloak in containers: https://www.keycloak.org/server/containers

* Apache Kafka (KRaft): https://kafka.apache.org/documentation/\#kraft

* Kafka UI (kafbat): https://ui.docs.kafbat.io/

* Docker Compose: https://docs.docker.com/compose/

## **Testing tools**

* TFLint: https://github.com/terraform-linters/tflint

* Checkov: https://www.checkov.io/

* Terratest: https://terratest.gruntwork.io/

* Conftest: https://www.conftest.dev/

* Infracost: https://www.infracost.io/docs/

# **Appendix B: Glossary**

| Term | Meaning |
| :---- | :---- |
| AMI | Amazon Machine Image — the OS install used to create an instance |
| ARN | Amazon Resource Name — a globally unique ID like arn:aws:iam::123:role/x |
| AZ | Availability Zone — one data center group inside a region |
| Backend | Where Terraform stores state |
| CIDR | Address range notation, e.g. 10.40.0.0/16 |
| Data source | A Terraform block that reads existing information |
| Drift | When real infrastructure differs from state/code |
| EBS | Elastic Block Store — virtual hard drives |
| EC2 | Elastic Compute Cloud — virtual servers |
| Egress / Ingress | Outbound / inbound traffic |
| EIP | Elastic IP — a permanent public IPv4 address |
| for\_each / count | Meta-arguments to create multiple copies |
| HCL | HashiCorp Configuration Language |
| IaC | Infrastructure as Code |
| IAM | Identity and Access Management |
| IGW | Internet Gateway |
| IMDS | Instance Metadata Service (v2 requires a token) |
| Instance profile | Wrapper that attaches an IAM role to an EC2 instance |
| KRaft | Kafka's built-in consensus mode (no ZooKeeper) |
| Launch template | Saved recipe for launching instances |
| Local | A named value computed inside a module |
| Module | A reusable folder of Terraform |
| NAT Gateway | Lets private subnets reach out to the internet |
| OIDC | OpenID Connect — the login protocol Kafka UI uses with Keycloak |
| Plan | The list of actions Terraform will take |
| Provider | Plugin that talks to one platform (AWS) |
| Realm | A Keycloak tenant holding users and clients |
| Resource | A thing Terraform creates and manages |
| Root module | The folder you run terraform in |
| Route table | Rules for where network traffic goes |
| SAN | Subject Alternative Name — the addresses a TLS certificate is valid for |
| Security group | Instance-level stateful firewall |
| SSM | AWS Systems Manager (Session Manager, Parameter Store) |
| Stack | A root module with its own state, deployed as a unit |
| State | Terraform's record of what it built |
| Subnet | A slice of a VPC in one AZ |
| tfvars | A file of variable values |
| User data | Script run at first boot |
| VPC | Virtual Private Cloud — your private network in AWS |

# **Appendix C: Quiz Answer Key**

## **Chapter 1: Infrastructure as Code**

**Q1 b.** Declarative means you describe the desired end state. Terraform computes the actions needed.

**Q2 c.** State records what exists and its real cloud IDs, so Terraform can compare it to your files.

**Q3 b.** The plan is the change order you approve before building.

**Q4 c.** IaC makes systems readable and reviewable, but you still must understand them.

**Q5 b.** Subnets are rooms inside the building (the VPC) where servers live.

## **Chapter 2: AWS Basics — The Services We Use**

**Q1 a.** A subnet is always in exactly one AZ. For multi-AZ you create several subnets.

**Q2 b.** The route to the IGW is what gives a subnet internet access.

**Q3 b.** A /24 leaves 8 bits free: 2^8 \= 256 addresses (a few are reserved by AWS).

**Q4 b.** The Keycloak certificate and Kafka UI redirect URLs embed the public IPs, so they must be known first.

**Q5 b.** The instance profile is the holder that attaches a role to an instance.

## **Chapter 3: How Terraform Works**

**Q1 b.** Init prepares the working directory; it never touches cloud resources.

**Q2 c.** Replacement can lose data; read the "forces replacement" reason.

**Q3 b.** Attribute references create implicit dependencies in the graph.

**Q4 b.** State can hold generated passwords and must be stored securely.

**Q5 b.** The AWS provider turns aws\_vpc into real AWS API calls.

## **Chapter 4: Setting Up and Your First Terraform Project**

**Q1 b.**

**Q2 b.** The provider reads the CLI's credentials automatically.

**Q3 b.** Local names only exist in your code; tags.Name is what AWS shows.

**Q4 c.** The lock file pins provider versions so teammates get the same ones.

**Q5 b.** Destroy removes what Terraform tracks in its state, nothing more.

## **Chapter 5: The Terraform Language — Every Keyword Explained**

**Q1 b.** for\_each iterates a map or set and keys each instance by name.

**Q2 c.** try returns the fallback when the first expression errors.

**Q3 c.** Command-line \-var overrides everything else.

**Q4 b.**

**Q5 b.** The plan is a fixed list of actions, so the number of copies must be decidable up front.

## **Chapter 6: Modules — Reusable Building Blocks**

**Q1 b.**

**Q2 c.** Providers belong in the root; child modules inherit them.

**Q3 a.**

**Q4 b.**

**Q5 b.** Zero copies means the module is skipped entirely.

## **Chapter 7: Types and Values — Strings, Numbers, Lists, Maps and Objects**

**Q1 b.** Only double quotes.

**Q2 b.**

**Q3 c.** Convert with tolist first.

**Q4 b.** Later maps win.

**Q5 c.**

## **Chapter 8: Variables, tfvars, Locals and Outputs in Depth**

**Q1 b.**

**Q2 b.** terraform.tfvars and \*.auto.tfvars load automatically.

**Q3 b.**

**Q4 b.**

**Q5 c.** Hence remote, encrypted state.

## **Chapter 9: Functions and Expressions Cookbook**

**Q1 b.**

**Q2 a.**

**Q3 c.**

**Q4 b.**

**Q5 b.**

## **Chapter 10: Resources in Depth — Required Settings and How to Read the Docs**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 11: Modules in Depth — Writing One From Scratch**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 12: What to Avoid — Terraform Anti-Patterns**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 13: Project Tour**

**Q1 c.**

**Q2 b.** The back-channel call stays inside the VPC, allowed by the SG-to-SG rule.

**Q3 b.** They document that network runs before apps.

**Q4 c.**

**Q5 b.** (Direct IDs are also supported as an override.)

## **Chapter 14: The VPC Module, Line by Line**

**Q1 b.**

**Q2 a.**

**Q3 b.** It needs the IGW to reach the internet on behalf of private subnets.

**Q4 b.**

**Q5 b.**

## **Chapter 15: The Security Groups Module, Line by Line**

**Q1 b.**

**Q2 c.**

**Q3 b.**

**Q4 b.**

**Q5 c.**

## **Chapter 16: The IAM Role and Elastic IP Modules**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 17: The Launch Template and EC2 Instance Modules**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 18: The Network Stack (Process 1), Line by Line**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 19: The Apps Stack (Process 2), Line by Line**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

## **Chapter 20: Deploy, Test and Tear Down — Step by Step**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 a.**

## **Chapter 21: Best Practices, Tips and Gotchas**

**Q1 b.** Others need \-var-file= (except \*.auto.tfvars).

**Q2 b.**

**Q3 b.**

**Q4 c.**

**Q5 b.**

## **Chapter 22: Making the Project Production-Ready**

**Q1 b.**

**Q2 b.**

**Q3 a.** The template is reused as-is — the benefit of a base module.

**Q4 b.**

**Q5 c.**

## **Chapter 23: Testing Terraform Projects**

**Q1 b.**

**Q2 b.**

**Q3 b.**

**Q4 b.**

**Q5 b.**

[image1]: <data:image/png;base64,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>