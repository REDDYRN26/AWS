# AWS EC2 - Complete Theory + Lab Guide (All Levels)

---

## 📚 LEVEL 1: AWS Associate (0-2 Years) 

### **THEORY TOPICS**

#### 1. EC2 Fundamentals
- **What is EC2?** - Elastic Compute Cloud, virtual machines in cloud
- **AMI (Amazon Machine Image)** - Template for launching instances
  - Public AMIs, Custom AMIs, Marketplace AMIs
- **Instance Types** - Computing power classifications
  - General Purpose (t3, m5, m6i)
  - Compute Optimized (c5, c6i)
  - Memory Optimized (r5, x1)
  - Storage Optimized (i3, h1)
- **Instance States**
  - Running, Stopped, Terminated, Rebooting
  - Difference between Stop vs Terminate
  - EBS volume behavior during each state

#### 2. Instance Lifecycle
- **Launch** - From AMI to running instance
- **Bootstrap** - User data scripts during startup
- **Connect to Instance**
  - SSH (Linux)
  - RDP (Windows)
  - EC2 Instance Connect
- **Stop vs Terminate**
  - Stop = Keep EBS volume, keep IP (elastic IP required)
  - Terminate = Delete everything, release IP
- **Reboot** - Restart without losing data

#### 3. Networking Basics
- **VPC (Virtual Private Cloud)** - Private network
- **Subnets** - Public vs Private
- **Security Groups** - Firewall at instance level
  - Inbound rules (allow traffic IN)
  - Outbound rules (allow traffic OUT)
  - Default allows all outbound
- **Key Pairs** - SSH key authentication
- **Elastic IP** - Static IP for instance
- **ENI (Elastic Network Interface)** - Virtual network card

#### 4. Storage
- **EBS (Elastic Block Store)** - Virtual hard drive
  - gp3, gp2 (General Purpose)
  - io1, io2 (High performance)
  - st1, sc1 (Throughput optimized)
- **Root Volume** - Where OS installed
- **Additional Volumes** - Data storage
- **Snapshots** - EBS backups
- **Instance Store** - Temporary storage (fast, ephemeral)

#### 5. Pricing Model
- **On-Demand** - Pay per hour, flexible
- **Reserved Instances** - Commit 1/3 years, discount 30-70%
- **Spot Instances** - Spare capacity, up to 90% discount
- **Savings Plans** - Flexible, compute family level discount
- **Pricing Factors**
  - Instance type
  - Region
  - OS (Linux cheaper than Windows)
  - Data transfer charges

#### 6. Monitoring & Management
- **CloudWatch** - Monitor CPU, network, disk
- **System Status Checks** - AWS infrastructure health
- **Instance Status Checks** - Operating system health
- **Auto Scaling** - Automatically add/remove instances
- **Load Balancing** - Distribute traffic
  - ELB (Elastic Load Balancer)
  - ALB (Application Load Balancer)
  - NLB (Network Load Balancer)

#### 7. Security Basics
- **IAM Roles** - Permissions for instances
- **Security Groups** - Network firewall
- **NACLs** - Subnet-level firewall
- **Encryption** - EBS volume encryption at rest
- **OS Security** - Patching, updates, SSH hardening

#### 8. Images & Templates
- **AMI Components** - Root volume, additional volumes
- **Creating Custom AMI** - From running instance
- **Launch Templates** - Reusable instance configs
- **Launch Configurations** - Deprecated (use templates instead)

---

### **LAB TOPICS - Hands-On Practice**

#### Lab 1: Create & Connect to EC2 Instance
```
1. Launch instance from public AMI (Amazon Linux 2)
2. Create/select key pair
3. Configure security group (allow SSH on port 22)
4. Launch instance
5. Wait for running state
6. Connect via SSH using public IP
7. Install software (Apache, nginx, etc.)
8. Verify connectivity from browser
```

#### Lab 2: Storage Management
```
1. Create instance with root volume
2. Attach additional EBS volume
3. Format and mount new volume
4. Create files on mounted volume
5. Create snapshot of volume
6. Create new volume from snapshot
7. Attach snapshot volume to another instance
8. Verify data persistence
```

#### Lab 3: Security Groups & Networking
```
1. Create security group with specific rules
2. Allow HTTP (80), HTTPS (443), SSH (22)
3. Deny all other traffic
4. Launch 2 instances in same VPC
5. Test connectivity between instances
6. Test external access
7. Modify security group rules (add/remove)
8. Verify changes take effect immediately
```

#### Lab 4: Elastic IP & Network Interfaces
```
1. Launch instance with default private IP
2. Allocate Elastic IP
3. Associate Elastic IP to instance
4. Verify IP persists after stop/start
5. Disassociate Elastic IP
6. Create additional ENI
7. Attach ENI to instance
8. Test multi-homed networking
```

#### Lab 5: AMI Creation
```
1. Launch and customize instance
2. Install applications
3. Configure settings
4. Create image from instance
5. Tag image with metadata
6. Launch new instance from custom AMI
7. Verify configurations applied
8. Delete custom AMI
```

#### Lab 6: User Data & Automation
```
1. Create user data script (bash)
   - Update system packages
   - Install Apache/Nginx
   - Start web server
   - Create test HTML page
2. Launch instance with user data
3. Wait for initialization
4. Verify web server running
5. Check logs: /var/log/cloud-init-output.log
6. Repeat with different user data scripts
```

#### Lab 7: Monitoring with CloudWatch
```
1. Launch instance with monitoring enabled
2. View CPU utilization graphs
3. View network metrics
4. Create custom CloudWatch alarm
   - Trigger on high CPU
   - Send SNS notification
5. Stress test instance (generate high CPU)
6. Verify alarm triggered
7. View logs in CloudWatch Logs
```

#### Lab 8: Instance Types & Performance
```
1. Launch different instance types (t3, m5, c5)
2. Compare CPU/Memory specs
3. Run performance test on each
4. Monitor performance metrics
5. Compare pricing for each type
6. Identify optimal instance type for workload
7. Document findings
```

#### Lab 9: Keypair Management
```
1. Create new keypair in AWS
2. Download .pem file securely
3. Change permissions: chmod 400 key.pem
4. SSH into instance with keypair
5. SSH without proper permissions (verify failure)
6. Import existing keypair
7. Delete keypair (verify instances still work)
8. Create new keypair for failed instances
```

#### Lab 10: Cost Optimization Basics
```
1. Launch instance as On-Demand
2. Calculate monthly cost
3. Compare with Reserved Instance pricing
4. Calculate savings
5. Use AWS Cost Calculator
6. Analyze cost by instance type, region, OS
7. Identify cost optimization opportunities
```

---

## 📘 LEVEL 2: AWS Professional (2-5 Years)

### **THEORY TOPICS** (Includes Level 1 + Advanced)

#### 1. Advanced Instance Types & Selection
- **vCPU Credits System** - t3 burstable instances
  - CPU credit accumulation
  - CPU credit usage
  - Baseline vs burst performance
  - T3 Unlimited mode
- **Instance Family Deep Dive**
  - When to use each generation (m5 vs m6i vs m7i)
  - Performance improvements across generations
  - Cost differences
- **Graviton Processors** - ARM-based instances (t4g, m7g, c7g)
  - Performance vs x86
  - Cost savings
  - Compatibility considerations

#### 2. Storage Advanced Topics
- **EBS Optimization** - Dedicated bandwidth for EBS
- **EBS Encryption**
  - At-rest encryption with KMS
  - Encrypting snapshots
  - Copying encrypted snapshots across regions
- **EBS Performance**
  - IOPS vs throughput
  - gp3 vs gp2 comparison
  - When to use io1/io2 vs gp3
- **Throughput Optimization**
  - EBS-optimized instances
  - GP3 custom IOPS/throughput tuning
- **Instance Store Storage**
  - Use cases (temporary caches, databases)
  - Data loss risk
  - Performance advantages
- **EBS Multi-Attach** (io1/io2)
  - Multiple instances attached to single volume
  - Cluster file systems

#### 3. Networking Advanced
- **VPC Peering** - Connect multiple VPCs
- **VPC Endpoints** - Private access to AWS services
  - Gateway endpoints (S3, DynamoDB)
  - Interface endpoints (EC2, SNS, etc.)
- **VPC Flow Logs** - Monitor network traffic
  - Accept/Reject logs
  - Analyzing logs for troubleshooting
- **Enhanced Networking**
  - SR-IOV (Single Root I/O Virtualization)
  - ENA (Elastic Network Adapter)
  - Performance improvements
- **Placement Groups**
  - Cluster - Low latency, high throughput
  - Partition - Distributed across racks
  - Spread - Individual racks for high availability

#### 4. High Availability & Disaster Recovery
- **Multi-AZ Architecture**
  - Automatic failover
  - When to use multi-AZ
  - Cost implications
- **Auto Scaling Groups**
  - Scaling policies (simple, step, target tracking)
  - Lifecycle hooks
  - Termination policies
  - Launch templates vs launch configurations
- **Load Balancers Deep Dive**
  - ALB (Layer 7, HTTP/HTTPS)
  - NLB (Layer 4, TCP/UDP, ultra-high performance)
  - ELB (Classic, legacy)
  - Target groups and health checks
- **Health Checks**
  - ELB health check configuration
  - Unhealthy instance handling
  - Connection draining

#### 5. Security Advanced
- **IAM Roles for EC2**
  - Assume role trust policy
  - Permission policies
  - Instance profiles
  - Temporary credentials rotation
- **VPC Security**
  - NACLs vs Security Groups
  - Security group chaining
  - Stateful connections
- **Encryption**
  - KMS key management
  - Volume encryption
  - Data encryption in transit (TLS/SSL)
- **Compliance & Auditing**
  - CloudTrail for audit logging
  - Config for compliance tracking
  - Security Hub

#### 6. Cost Management & Optimization
- **Reserved Instances**
  - Standard vs Convertible
  - Capacity reservations
  - Regional vs Zonal RIs
- **Savings Plans**
  - Compute Savings Plans
  - EC2 Instance Savings Plans
  - Blended vs On-Demand rates
- **Spot Instances**
  - Spot fleets
  - Spot interruption handling
  - On-demand fallback
  - Cost savings analysis
- **Cost Allocation Tags** - Track costs by department/project
- **Compute Optimizer** - Right-sizing recommendations

#### 7. Automation & Infrastructure as Code
- **CloudFormation**
  - Templates (JSON/YAML)
  - Stack management
  - Stack policies
- **Terraform** - Multi-cloud IaC
- **AWS Systems Manager**
  - Parameter Store for configs
  - Patch Manager for OS updates
  - Run Command for automation

#### 8. Performance Optimization
- **CPU Credit Tracking** - t-series instances
- **Enhanced Networking** - SR-IOV, ENA performance
- **Nitro System** - Newer instance generation benefits
- **Memory Optimization** - Large instances for databases
- **I/O Optimization** - io1/io2 for high IOPS workloads

#### 9. Licensing & Compliance
- **License Portability** - Bring your own license (BYOL)
- **Microsoft License Terms** - Dedicated hosts, Windows licensing
- **Compliance Requirements** - PCI-DSS, HIPAA, GDPR
- **Dedicated Hosts vs Dedicated Instances**

#### 10. Monitoring & Troubleshooting Advanced
- **CloudWatch Deep Dive**
  - Custom metrics
  - Detailed monitoring
  - Metric math
  - Composite alarms
- **System Manager Session Manager** - SSH alternative
- **VPC Flow Logs Analysis** - Debugging network issues
- **CloudTrail** - API audit logging

---

### **LAB TOPICS - Hands-On Practice**

#### Lab 1: Auto Scaling Group with Load Balancer
```
1. Create Launch Template with user data
2. Create Application Load Balancer
3. Create Target Group
4. Configure Auto Scaling Group (ASG)
   - Min capacity: 1
   - Desired: 2
   - Max: 4
5. Configure scaling policy
   - Target CPU utilization: 50%
6. Verify instances launching in ASG
7. Stress test to trigger scaling
8. Verify new instances added
9. Verify load balancer distributes traffic
10. Monitor CloudWatch metrics
```

#### Lab 2: Multi-AZ Architecture
```
1. Create VPC with 3 subnets (3 AZs)
2. Launch instances in each AZ
3. Create Network Load Balancer across AZs
4. Configure health checks
5. Test failover by stopping instance
6. Verify traffic redirects to healthy instance
7. Monitor availability zone distribution
8. Test cross-AZ latency (optional)
```

#### Lab 3: VPC Endpoints
```
1. Create S3 bucket
2. Upload test file to S3
3. Launch instance without public IP (private)
4. Try accessing S3 from instance (fails)
5. Create Gateway VPC Endpoint for S3
6. Update route tables
7. Access S3 from instance (succeeds)
8. Compare: with NAT Gateway vs Gateway Endpoint (cost)
```

#### Lab 4: Placement Groups
```
1. Launch instances in Cluster Placement Group
2. Launch instances in Spread Placement Group
3. Run network performance test
4. Compare latency/throughput
5. Verify physical placement (AWS Console)
6. Test failover within placement groups
7. Document performance differences
```

#### Lab 5: Security Groups & NACLs
```
1. Create VPC with subnet
2. Create restrictive NACL
3. Create security group
4. Launch instance
5. Test connectivity with different rules
6. Compare Security Group vs NACL behavior
7. Understand stateful vs stateless
8. Troubleshoot connection issues
```

#### Lab 6: IAM Roles for EC2
```
1. Create IAM role with S3 access policy
2. Create instance profile
3. Launch instance with instance profile
4. Access S3 from instance (should work)
5. Check temporary credentials in instance
   - cat /proc/urn/creds file
6. Verify credentials auto-rotate
7. Test with different IAM policies
8. Troubleshoot access denied errors
```

#### Lab 7: CloudFormation Stack
```
1. Write CloudFormation template (YAML)
   - VPC
   - Subnet
   - Security Group
   - EC2 instance
   - EBS volume
   - Outputs
2. Create stack using template
3. Verify all resources created
4. Modify template (e.g., change instance type)
5. Update stack
6. Delete stack and verify cleanup
7. Practice stack drift detection
```

#### Lab 8: Cost Analysis & Savings Plans
```
1. Launch multiple instance types
2. Use AWS Cost Explorer
3. Analyze costs by instance type
4. Calculate RI savings
5. Calculate Spot savings
6. Compare On-Demand vs Reserved vs Spot
7. Implement Savings Plan
8. Track actual vs projected savings
```

#### Lab 9: Systems Manager for Patching
```
1. Create custom AMI with outdated packages
2. Launch instance from AMI
3. Set up Systems Manager Patch Manager
4. Define patch groups
5. Create patch baseline
6. Run patch scan
7. Apply patches using maintenance window
8. Verify patches applied
9. Reboot if necessary
```

#### Lab 10: VPC Flow Logs Analysis
```
1. Enable VPC Flow Logs
2. Launch instance
3. Generate traffic (SSH, HTTP, etc.)
4. Generate blocked traffic
5. Query flow logs in CloudWatch Logs
6. Filter for accepted/rejected traffic
7. Troubleshoot connection issues using logs
8. Identify suspicious traffic patterns
```

---

## 🎓 LEVEL 3: AWS Solutions Architect / Senior (5+ Years)

### **THEORY TOPICS** (Includes Level 1 + 2 + Expert)

#### 1. Complex Architecture Patterns
- **High Performance Computing (HPC)**
  - Cluster Compute instances
  - Enhanced networking
  - Placement groups (Cluster)
  - Low latency requirements
- **Batch Processing**
  - Spot instances for cost
  - Auto Scaling for variable workloads
  - Job prioritization
- **Real-Time Analytics**
  - Memory-optimized instances
  - High throughput requirements
  - Network optimization

#### 2. Enterprise Networking
- **Site-to-Site VPN**
  - Customer Gateway, Virtual Private Gateway
  - BGP dynamic routing
  - Failover design
- **AWS Direct Connect**
  - Dedicated network connection
  - Consistent network performance
  - Integration with multiple VPCs
- **Transit Gateway**
  - Hub-and-spoke architecture
  - Multi-VPC/On-premises connectivity
  - Centralized routing

#### 3. Disaster Recovery & Business Continuity
- **RPO/RTO Analysis**
  - Recovery Point Objective
  - Recovery Time Objective
- **DR Strategies**
  - Backup & Restore
  - Pilot Light
  - Warm Standby
  - Hot Standby/Multi-Site Active-Active
- **Cross-Region Replication**
  - AMI copying
  - Snapshot copying
  - Database replication

#### 4. Container & Microservices
- **ECS (Elastic Container Service)**
  - EC2 launch type
  - Fargate launch type
  - Container agent
  - Task definitions
- **EC2 Cluster Optimization**
  - Right-sizing for container workloads
  - Container density
  - Bin packing strategies
- **Container Registry Integration**
  - ECR (Elastic Container Registry)
  - Image caching
  - Registry authentication

#### 5. Advanced Security Architecture
- **Defense in Depth**
  - Multiple layers of security
  - Security groups, NACLs, WAF, GuardDuty
- **Zero Trust Architecture**
  - Assume breach mentality
  - Continuous validation
  - Micro-segmentation with security groups
- **Compliance Automation**
  - Config rules
  - Automated remediation
  - Audit trails (CloudTrail)
- **Encryption Strategy**
  - KMS key hierarchy
  - Envelope encryption
  - Field-level encryption
- **Hardware Security Modules**
  - CloudHSM
  - Key management
  - Compliance requirements (PCI-DSS, HIPAA)

#### 6. Advanced Performance Tuning
- **CPU Affinity** - Bind processes to specific cores
- **NUMA Optimization** - Non-uniform memory access
- **Cache Optimization** - L1/L2/L3 cache efficiency
- **System Tuning Parameters**
  - TCP window scaling
  - Buffer sizes
  - Network stack tuning
- **Benchmark Analysis**
  - SPECCpu, SPECrate
  - Custom workload benchmarking
  - Performance baselines

#### 7. Financial & Strategic Optimization
- **Reserved Capacity Planning**
  - Forecast-based commitment
  - Blended rates
  - Regional optimization
- **Spot Instance Strategy**
  - Interruption handling (Kubernetes, batch jobs)
  - Cost optimization (up to 90% savings)
  - Diversification across instance types
- **Total Cost of Ownership (TCO)**
  - On-premises vs AWS comparison
  - Hidden cloud costs
  - Egress charges optimization
- **FinOps Practices**
  - Chargeback models
  - Cost allocation by department
  - Cost forecasting

#### 8. Organizational & Governance
- **Multi-Account Strategy**
  - AWS Organizations
  - Service Control Policies (SCPs)
  - Cross-account IAM roles
- **Tagging Strategy**
  - Mandatory tags
  - Cost allocation tags
  - Automation tags
- **Change Management**
  - Canary deployments
  - Blue-green deployments
  - Rollback procedures
- **Audit & Compliance**
  - Evidence collection
  - Compliance reporting
  - Remediation workflows

#### 9. Hybrid & On-Premises Integration
- **AWS Outposts** - AWS hardware on-premises
- **VMware Cloud on AWS** - VMware workloads
- **Data Residency Requirements** - Geographic compliance
- **Network Consistency** - Hybrid network design

#### 10. Emerging Technologies
- **Graviton Processors**
  - Performance vs cost
  - Migration considerations
  - Application compatibility
- **AWS Nitro System**
  - Hypervisor improvements
  - Bare metal capabilities
  - Security benefits
- **GPU & Accelerator Instances**
  - Deep learning instances
  - Graphics rendering instances
  - Financial modeling instances

---

### **LAB TOPICS - Hands-On Practice**

#### Lab 1: Multi-Region Active-Active Architecture
```
1. Design architecture in 2 regions
   - Launch instances in each region
   - Create EBS volumes in each
   - Set up cross-region replication
2. Create Route53 health checks
3. Configure geolocation routing
4. Test failover between regions
5. Verify data consistency
6. Measure RPO/RTO
7. Document recovery process
8. Calculate cost implications
```

#### Lab 2: Transit Gateway with Multiple VPCs
```
1. Create 3 VPCs with different CIDR blocks
2. Launch instances in each VPC
3. Create Transit Gateway
4. Attach all VPCs to TGW
5. Configure route tables
6. Configure DNS (peering DNS)
7. Test connectivity between VPCs
8. Implement inspection via Security appliance
9. Monitor traffic flow
10. Troubleshoot connectivity issues
```

#### Lab 3: ECS Cluster on EC2 with Auto Scaling
```
1. Create ECS cluster on EC2
2. Create task definition
3. Create service with load balancer
4. Configure service-level scaling
5. Configure cluster-level scaling (ASG)
6. Monitor task placement
7. Test failover
8. Analyze resource utilization
9. Optimize instance types
10. Implement cost optimization
```

#### Lab 4: Disaster Recovery Drill
```
1. Set up primary region with application
2. Replicate to DR region
3. Create automated snapshot schedule
4. Copy snapshots to DR region
5. Create AMI in DR region
6. Launch stack in DR region (keep stopped)
7. Perform DR drill
   - Failover to DR region
   - Verify application functionality
   - Measure RTO/RPO
   - Document issues
8. Failback to primary
9. Document runbook
```

#### Lab 5: Security Hub & Compliance Automation
```
1. Enable Security Hub
2. Enable various compliance standards
   - CIS AWS Foundations Benchmark
   - PCI DSS
3. Launch EC2 with security issues
4. Verify findings generated
5. Create Config rules for remediation
6. Auto-remediate findings
7. Track compliance posture over time
8. Generate compliance reports
```

#### Lab 6: Cross-Account Access & Audit
```
1. Create 2 AWS accounts
2. Set up cross-account IAM role
3. Launch EC2 in account A
4. Access from account B
5. Enable CloudTrail in both accounts
6. Verify audit logs
7. Implement centralized logging
8. Analyze access patterns
9. Detect suspicious activity
10. Implement automated alerts
```

#### Lab 7: Performance Benchmark & Tuning
```
1. Launch performance-optimized instance
2. Launch network-optimized instance
3. Install benchmarking tools (sysbench, iperf)
4. Run CPU benchmarks
5. Run network benchmarks
6. Run disk I/O benchmarks
7. Compare performance metrics
8. Tune system parameters
9. Re-benchmark after tuning
10. Document performance improvements
```

#### Lab 8: Spot Instance Strategy with Interruption Handling
```
1. Create Spot Fleet with diversified instance types
2. Configure on-demand fallback
3. Implement interruption handler
4. Deploy application with graceful shutdown
5. Trigger interruption (optional, AWS schedule)
6. Verify automatic fallback
7. Monitor cost savings
8. Calculate actual vs projected savings
9. Implement notifications
```

#### Lab 9: AWS Systems Manager Fleet Management
```
1. Launch multiple instances
2. Tag instances appropriately
3. Create Systems Manager documents
4. Run command across fleet
   - OS updates
   - Configuration changes
   - Application deployment
5. Schedule maintenance window
6. Execute automation runbook
7. Monitor compliance state
8. Generate inventory reports
9. Track patch compliance
```

#### Lab 10: Hybrid Network with Site-to-Site VPN
```
1. Set up simulated on-premises environment (EC2 in different VPC)
2. Create VPN gateway
3. Create customer gateway
4. Establish VPN connection
5. Configure routing
6. Test connectivity between on-premises & AWS
7. Monitor VPN connection metrics
8. Implement redundancy (dual VPN)
9. Test failover
10. Document network diagram & runbook
```

---

## 📊 COMPARISON TABLE

| Topic | Associate | Professional | Senior |
|-------|-----------|--------------|--------|
| **Instance Types** | Basic selection | Advanced tuning (T-unlimited, vCPU credits) | Benchmark & optimization |
| **Networking** | Security groups, subnets | VPC endpoints, peering, flow logs | Transit Gateway, Direct Connect |
| **Storage** | Basic EBS | EBS encryption, snapshots | EBS optimization, multi-attach |
| **HA/DR** | Basic concepts | Multi-AZ, ASG, RIs | Multi-region, cross-account, RTO/RPO |
| **Security** | Security groups, IAM basics | Compliance, KMS, audit logging | Zero trust, defense in depth |
| **Cost** | On-Demand, RIs, Spot basics | Savings Plans, Compute Optimizer | FinOps, TCO analysis, chargeback |
| **Automation** | User data scripts | CloudFormation, Terraform | Config, Systems Manager fleet |

---

## ✅ LEARNING PATH RECOMMENDATION

### **Week 1-2: Associate Level**
- Study theory topics 1-8
- Complete labs 1-10
- Practice on AWS Free Tier

### **Week 3-6: Professional Level**
- Study advanced topics
- Complete professional labs 1-10
- Build real-world projects

### **Week 7-10: Senior Level**
- Study enterprise topics
- Complete expert labs 1-10
- Design complex architectures
- Get hands-on with tools (Terraform, CloudFormation)

---

## 🎯 Interview Preparation Tips

### **For Associate Level Interview:**
1. Focus on fundamentals
2. Know difference between terms (Stop vs Terminate, RIs vs Spot)
3. Practice creating instances and managing them
4. Understand security groups deeply

### **For Professional Level Interview:**
1. Design multi-tier architectures
2. Explain Auto Scaling strategy
3. Cost optimization examples
4. Troubleshooting scenarios

### **For Solutions Architect Interview:**
1. Design enterprise-scale architecture
2. Explain business requirements → AWS solution
3. DR/BC strategy
4. Multi-region considerations
5. Compliance & governance

---

## 📚 ADDITIONAL RESOURCES

- **Official:** AWS EC2 documentation
- **Hands-on:** AWS free tier (750 hours/month)
- **Certification:** AWS Certified Solutions Architect, AWS Certified DevOps Engineer
- **Practice Exams:** A Cloud Guru, Linux Academy
- **Real-world:** Build projects on AWS

---

**Good luck with your AWS learning journey!** 🚀
