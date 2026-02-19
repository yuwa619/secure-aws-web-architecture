## Architecture Overview

**Region:** eu-west-2 (London)
**VPC:** Default VPC

**Components:**

* Bastion Host (public subnet) for secure SSH access
* Two EC2 Web Servers (private subnet, no public IPs)
* Application Load Balancer (internet-facing)
* NAT Gateway for outbound internet access from private instances
* Ansible controller using SSH ProxyJump

📷 Architecture diagram available in `/diagrams`

---

## Security Design

* Web servers have **no public IP addresses**
* SSH access allowed **only via bastion host**
* SSH keys remain on the Ansible controller (never copied to bastion)
* Web servers accept HTTP traffic **only from the ALB**
* Security groups follow least-privilege principles

---

## Ansible Automation

Ansible was used to:

* Install NGINX on private EC2 instances
* Enable and start the service
* Deploy a dynamic HTML page using Jinja2 templates

The HTML page displays:

* Hostname
* Private IP address

This confirms successful load balancing through the ALB.

---

## NAT Gateway Black Hole Issue

During deployment, Ansible hung while installing NGINX.

**Root cause:**
Private subnet route table lacked a `0.0.0.0/0` route to a NAT Gateway.
Outbound traffic had no path and was silently dropped (black hole).

**Fix:**

* Created a NAT Gateway in the public subnet
* Updated private route table to route outbound traffic via the NAT Gateway

After this fix, Ansible completed successfully.

---

## Verification

* Ansible `ping` module succeeded on all hosts
* ALB target group showed both instances as healthy
* Browser access via ALB DNS alternated between servers
* Direct access to private IPs from the internet was not possible

📷 Verification screenshots available in `/screenshots`

---

## Technologies Used

* AWS EC2
* Application Load Balancer
* NAT Gateway
* Security Groups
* Ansible
* SSH ProxyJump
* NGINX
* Amazon Linux

---

## Author

**Martins Obaseki**
Cloud / DevOps Engineer 
