# ⚖️ AWS EC2 Lab – Application Load Balancer (ALB) with Target Groups

In this lab, I deployed a complete **Application Load Balancer (ALB)** setup for **two private EC2 instances** using **Terraform**.
I learned how to distribute traffic between multiple applications (on ports **80** and **8080**) using **path-based routing rules**, created distinct **target groups** for each app, and verified that both endpoints respond correctly via the ALB DNS.

---

## 📋 Lab Overview

**Goal:**

* Use Terraform to deploy a VPC, subnets, and EC2 instances.
* Configure an **Application Load Balancer (ALB)** with path-based routing.
* Create and associate **target groups** for private instances.
* Validate connectivity using ALB DNS with `/` and `/test` paths.

**Learning Outcomes:**

* Understand how ALBs route traffic using **listeners and rules**.
* Differentiate between **target groups** and **listeners**.
* Use **Terraform** to automate networking and EC2 deployments.
* Test path-based routing for multiple backend applications.

---

## 🛠 Step-by-Step Journey

### Step 1 – Log In & Set Region

Logged in using provided credentials.
Set AWS region to **US East (N. Virginia)** → `us-east-1`.

✅ Environment ready.

---

### Step 2 – Initialize Terraform Stack

Navigate to the Terraform code directory and initialize:

```bash
cd /app/terraform_files/stack
terraform init
terraform plan
terraform apply -auto-approve
```

Terraform provisions the following:

* **VPC** with 3 public and 3 private subnets
* **2 EC2 instances:**

  * `app1` → runs on port **80**
  * `app2` → runs on port **8080**
* **Security group** allowing inbound **HTTP (80)** and **8080**

✅ Infrastructure successfully deployed.

---

### Step 3 – Review the Concept: When to Use an ALB

**Application Load Balancers (ALBs)** are ideal for:

* Distributing **HTTP/HTTPS** traffic
* Path-based and host-based routing
* Load balancing across multiple targets in different AZs

✅ Correct use case: **Load balancing HTTP and HTTPS traffic.**

---

### Step 4 – Create Target Group for `app1`

1. Go to **EC2 → Target Groups → Create target group**
2. Configuration:

| Setting               | Value     |
| --------------------- | --------- |
| **Target type**       | Instances |
| **Name**              | `app1`    |
| **Protocol**          | HTTP      |
| **Port**              | 80        |
| **VPC**               | `lab-vpc` |
| **Health check path** | `/`       |
| **Success codes**     | `200`     |

3. Register the **`app1` EC2 instance**.

✅ Target group `app1` created successfully.

---

### Step 5 – Create Target Group for `app2`

1. Go to **Target Groups → Create target group**
2. Configuration:

| Setting               | Value     |
| --------------------- | --------- |
| **Target type**       | Instances |
| **Name**              | `app2`    |
| **Protocol**          | HTTP      |
| **Port**              | 8080      |
| **VPC**               | `lab-vpc` |
| **Health check path** | `/test`   |
| **Success codes**     | `200`     |

3. Register the **`app2` EC2 instance**.

✅ Target group `app2` created successfully.

---

### Step 6 – Create Application Load Balancer

1. Go to **EC2 → Load Balancers → Create Load Balancer → Application Load Balancer**
2. Configure:

| Setting            | Value            |
| ------------------ | ---------------- |
| **Name**           | `kk-lab`         |
| **Scheme**         | Internet-facing  |
| **VPC**            | `lab-vpc`        |
| **Subnets**        | 3 public subnets |
| **Security group** | `allow_TLS`      |
| **Listener**       | HTTP on port 80  |

3. Default action: Forward to **target group app1**.

✅ Load Balancer `kk-lab` created and status became **Active**.

---

### Step 7 – Test the Load Balancer

Copy the **DNS name** of the ALB (e.g., `kk-lab-123456.us-east-1.elb.amazonaws.com`).
Open in browser or run:

```bash
curl http://<ALB_DNS_NAME>
```

✅ Output from `app1` confirmed on `/` route.

---

### Step 8 – Add Listener Rule for Path-Based Routing

1. Navigate to **kk-lab → Listeners → HTTP :80 → Manage Rules**
2. Add a new rule:

   * **Condition:** Path is `/test`
   * **Action:** Forward to target group `app2`
3. Move the new rule above the default one (priority = 1).

✅ Listener rule for `/test` added successfully.

---

### Step 9 – Verify Path Routing

From the terminal:

```bash
# Default route → app1
curl http://<ALB_DNS_NAME>

# /test route → app2
curl http://<ALB_DNS_NAME>/test
```

✅ Both responses successful:

* `/` → handled by **app1**
* `/test` → handled by **app2**

---

## 🏁 End of Lab

### ✅ Key Actions Summary

| Task                 | Action                                            |
| -------------------- | ------------------------------------------------- |
| Initialize Terraform | `terraform init && terraform apply -auto-approve` |
| Create Target Groups | app1 (80) & app2 (8080)                           |
| Create ALB           | Internet-facing `kk-lab`                          |
| Add Listener         | Port 80 → HTTP                                    |
| Add Routing Rules    | `/` → app1, `/test` → app2                        |
| Test                 | `curl` ALB DNS name + `/test`                     |

---

### 💡 Notes / Tips

* **Application Load Balancer (ALB)** operates at **Layer 7 (HTTP/HTTPS)** of the OSI model.
* Ideal for **stateless** web apps that require intelligent routing.
* Path-based routing enables multiple microservices behind one ALB.
* Always ensure **health checks** return **200 OK** responses for stability.
* **Terraform** simplifies multi-resource creation across VPC, subnets, and EC2.

---

### ✅ References

* [AWS Application Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
* [Target Groups and Health Checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
* [Terraform AWS ALB Resource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)
* [Path-Based Routing Example](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html)
