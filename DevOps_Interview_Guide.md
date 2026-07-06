# DevOps Interview Guide
### Candidate: 8 Years Experience | Accenture Mock Interview

---

## 1. Interview Process – How to Conduct

### Total Duration: 60–75 minutes

- **0–5 min:** Candidate introduction
- **5–15 min:** Project deep-dive – real work, team size, roles
- **15–35 min:** Core technical deep questions (Kubernetes, Docker, CI/CD)
- **35–50 min:** Cloud + IaC + Scripting (AWS, Terraform, Python)
- **50–65 min:** Real-time scenario-based shock questions (troubleshooting)
- **65–75 min:** Behavioral, on-call stories, questions from candidate

### Interview Tone

- Start friendly – 2 mins of casual conversation
- Gradually increase depth: easy → medium → hard → shock
- After every answer, follow up with "why" or "what if"
- Use silence as a tool – wait 10 seconds, candidate panics and gives real answer
- Cross-question: "Do you decide this directly, or how does the team decide?"

### Profile Gap Check (IMPORTANT)

Resume shows 5 years of GCP/AWS experience, but you need to treat the candidate as 8 years experienced. So:

- Ask about older projects (2017–2020 period) – how were they handled?
- Ask about legacy migration experience – on-prem to cloud
- Ask about leadership/mentoring experience – 8 years implies junior team lead expectation

---

## 2. Introduction & Project Deep-Dive Questions

**Q1.** Tell me about yourself and walk me through your 8 years journey in IT.
> *Answer key: Clear timeline expected – how did they transition from Linux/System admin to DevOps. Version jumps, role changes must be clear.*

**Q2.** Explain your current project architecture end-to-end. How is the application deployed, how does traffic flow?
> *Answer key: Ask them to draw it (whiteboard/paper). User → LB → Ingress → Service → Pod → DB. Red flag if they can't explain.*

**Q3.** How many tickets do you close daily in your current role? How do you decide priority?
> *Answer key: Real number expected (5–15/day typical). P1/P2/P3 definitions should be clear.*

**Q4.** What is your team size? Do you work solo or in pair/team?
> *Answer key: Team structure, reporting manager, onshore-offshore model should be explained.*

**Q5.** Tell me about your most challenging production incident – what happened, how did you fix it, what was the RCA?
> *Answer key: STAR format – Situation, Task, Action, Result. Specific metrics (downtime minutes, users affected) must be mentioned.*

**Q6.** Your resume mentions GCP to AWS migration. How many applications? What strategy did you take?
> *Answer key: Lift-and-shift vs re-architect choice should be explained. Tools: Velostrata, CloudEndure, or custom scripts.*

---

## 3. Kubernetes & OpenShift – Deep Real-time Questions

> *Candidate claims K8s strength. Don't ask basics. Go directly into depth.*

**Q1.** A pod is in CrashLoopBackOff. There are no logs. How do you debug step-by-step?
> *Answer key: `kubectl describe pod` → check events, `kubectl logs --previous`, exec into init container, check liveness probe, resource limits, image pull issues, missing secrets/configmap.*

**Q2.** A Node went into NotReady state in production. 50 pods are on it. What do you do first?
> *Answer key: `kubectl describe node` → conditions (DiskPressure/MemoryPressure), kubelet logs, drain node, cordon, check network plugin (CNI), disk space, then replace node.*

**Q3.** You're doing a rolling update in Deployment. New pods are not starting – stuck in Pending. Reason?
> *Answer key: Node resource insufficient, PVC binding issue, taints/tolerations mismatch, image pull secret missing, PodDisruptionBudget blocking, affinity rules conflict.*

**Q4.** What is the difference between StatefulSet and Deployment? Give a real use case where you used StatefulSet.
> *Answer key: Stable network identity, ordered deployment, persistent storage per pod. Use cases: Kafka, MongoDB, Elasticsearch, Zookeeper cluster.*

**Q5.** How does pod-to-pod communication work across different namespaces? How have you implemented Network Policy?
> *Answer key: ClusterIP service FQDN: `svc.namespace.svc.cluster.local`. NetworkPolicy with podSelector, namespaceSelector, ingress/egress rules. Requires Calico/Cilium CNI.*

**Q6.** You configured HPA. CPU crossed 80% but pods are not scaling. Why?
> *Answer key: metrics-server not installed/unhealthy, resource requests not set on pod (HPA needs requests), HPA min/max limits, stabilization window, cooldown period.*

**Q7.** How do you do cluster upgrades in production? How do you ensure zero downtime?
> *Answer key: Control plane first → worker nodes one-by-one. Drain → upgrade kubelet → uncordon. Configure PDB. Blue-green or canary cluster approach. etcd backup mandatory before.*

**Q8.** Tell me 5 key differences between OpenShift and vanilla Kubernetes.
> *Answer key: Routes vs Ingress, SCC (Security Context Constraints) vs PSP, BuildConfig/ImageStream, `oc` CLI extra commands, integrated OAuth, native Operator Hub.*

**Q9.** In ROSA cluster, who manages the control plane – you or AWS?
> *Answer key: AWS + Red Hat jointly manage the control plane. Customer manages only worker nodes and workloads. STS mode recommended for auth.*

**Q10.** You say containers should not run as root. How do you enforce this?
> *Answer key: SecurityContext: `runAsNonRoot: true`, `runAsUser: 1000`, `readOnlyRootFilesystem`, capabilities drop ALL. OpenShift SCC restricted-v2. PodSecurityStandards (baseline/restricted).*

**Q11.** How do you take etcd backup? How do you restore it? Have you actually done it in real life?
> *Answer key: `etcdctl snapshot save`, encrypt, store in S3 with versioning. Restore: stop kube-apiserver, `etcdctl snapshot restore`, update manifests, restart. Test monthly.*

**Q12.** Secrets are stored in plaintext in etcd by default. How do you encrypt them?
> *Answer key: EncryptionConfiguration at rest – aescbc/kms provider. Sealed Secrets, External Secrets Operator with AWS Secrets Manager/Vault integration.*

---

## 4. Docker – Deep Questions

**Q1.** Why do you use multi-stage builds in Dockerfile? Give an example.
> *Answer key: Reduce final image size, separate build dependencies from runtime. Example: Go/Java build stage → Alpine runtime stage. 1GB → 20MB.*

**Q2.** What are the isolation-level differences between Docker containers and VMs?
> *Answer key: Container shares kernel, uses namespaces + cgroups for isolation. VM has its own kernel, hypervisor overhead. Container startup: seconds vs minutes.*

**Q3.** How do you optimize image size? How have you done it in production?
> *Answer key: Alpine/distroless base, multi-stage, `.dockerignore`, combine RUN commands, remove apt cache, specific COPY instead of `COPY . .`*

**Q4.** Containers should not run as root. How do you configure that in Dockerfile?
> *Answer key: `USER 1000` or `USER nonroot`, create user in Dockerfile: `RUN adduser`. Combine with SecurityContext in K8s.*

**Q5.** How do you scan Docker images for vulnerabilities? How do you integrate it in the pipeline?
> *Answer key: Trivy, Clair, Anchore, Snyk. Scan in Jenkins stage, fail build on CVE threshold, signed images with cosign.*

---

## 5. AWS & ROSA – Cloud Deep Questions

**Q1.** EKS vs ROSA vs self-managed K8s on EC2 – why did you choose ROSA?
> *Answer key: ROSA – Red Hat support, OpenShift features, enterprise security (SCC), integrated CI/CD, AWS native integration. EKS – AWS native, cheaper. Self-managed – full control, high effort.*

**Q2.** IAM role vs IAM user – how do you give AWS permissions to a pod?
> *Answer key: IRSA (IAM Roles for Service Accounts) – OIDC provider, trust policy, annotation on SA. Better than node role (least privilege).*

**Q3.** How are VPC CNI and Calico different? What does ROSA use?
> *Answer key: VPC CNI – pod gets VPC IP directly, ENI limits per node. Calico – overlay, network policies. ROSA uses OVN-Kubernetes by default.*

**Q4.** Data was accidentally deleted from an S3 bucket. How do you recover?
> *Answer key: If versioning enabled – restore previous version. MFA delete, Object Lock for compliance. Cross-region replication for DR.*

**Q5.** In production, Load Balancer health check is failing but pods are running. How do you troubleshoot?
> *Answer key: Target group health check path, port, protocol must match. Security group rules LB → pod port. Readiness probe vs LB check mismatch. Check ALB ingress controller logs.*

**Q6.** How do you optimize AWS cost for Kubernetes workloads?
> *Answer key: Spot instances for non-critical workloads, Karpenter/Cluster Autoscaler, right-sizing with VPA, reserved instances for baseline, S3 lifecycle, cleanup unused EBS volumes.*

---

## 6. Terraform & IaC – Deep Questions

**Q1.** Your Terraform state file got corrupted. How do you recover?
> *Answer key: Remote backend (S3 + DynamoDB lock) with versioning enabled – restore previous version. `terraform state pull/push`. Worst case: `terraform import` each resource.*

**Q2.** `terraform plan` shows destruction of a production resource. What do you do?
> *Answer key: STOP. Don't apply. Check state drift – someone modified manually. `terraform refresh`, compare, use `lifecycle { prevent_destroy = true }` for critical resources.*

**Q3.** Module vs Workspace – why and when do you use them?
> *Answer key: Modules – reusable components (VPC module, EKS module). Workspaces – same config for different environments (dev/stg/prod). Better approach: separate state files per env.*

**Q4.** Why do you use both Terraform and Ansible? Isn't there overlap?
> *Answer key: Terraform – provisioning (infra creation). Ansible – configuration management (post-provisioning – package install, config files). Complementary, not replacement.*

**Q5.** How do you handle sensitive variables (DB password) in Terraform?
> *Answer key: `sensitive = true` flag, AWS Secrets Manager/Vault data source, never commit tfvars, use `TF_VAR_` env variables, encrypted backend.*

---

## 7. CI/CD – Jenkins, ArgoCD, Helm, Git

**Q1.** How do you do automatic rollback in a Jenkins pipeline if deployment fails?
> *Answer key: try-catch-finally block, `post { failure { rollback stage } }`, `kubectl rollout undo`, `helm rollback <release> <revision>`, notify via Slack/email.*

**Q2.** Why both ArgoCD and Jenkins? What is GitOps?
> *Answer key: Jenkins – CI (build, test, push image). ArgoCD – CD (pull-based from Git). GitOps – Git is single source of truth, declarative, auto-sync, self-heal.*

**Q3.** How do you override `values.yaml` in Helm chart for multi-environment?
> *Answer key: Separate `values-dev.yaml`, `values-prod.yaml`. `helm install -f values-prod.yaml`. Subcharts, global values, `--set` for overrides.*

**Q4.** Difference between Helm 2 and Helm 3? Why was Tiller removed?
> *Answer key: Helm 3 – no Tiller (security issue – cluster-admin), release stored in namespace Secret, 3-way merge, library charts, improved CRD support.*

**Q5.** A Git merge conflict came up during a production hotfix. How do you handle it?
> *Answer key: `git stash`, create hotfix branch from main/master, fix, merge to main AND develop, tag release, cherry-pick if needed.*

**Q6.** How do you set up Jenkins master-slave architecture? How are jobs assigned to slaves?
> *Answer key: Labels on slaves, `agent { label 'docker-build' }`. SSH/JNLP agents. Kubernetes plugin for dynamic agents. Scale based on queue.*

**Q7.** What branching strategy do you follow – GitFlow vs Trunk-based?
> *Answer key: GitFlow – develop, feature, release, hotfix, main. Trunk-based – short-lived branches, feature flags, continuous deployment. Depends on team size / release cadence.*

---

## 8. Shock Scenario Questions – Test the Candidate

> **IMPORTANT:** These questions are not about the answer, they are about observing the thinking process. See if the candidate can approach it in a structured way without panicking.

**Q1.** It's 3 AM. A PagerDuty call comes in. Production website is down. You log in. On the monitoring dashboard, all pods are in CrashLoopBackOff. What is your first step?
> *Answer key: Don't panic. 1) Acknowledge alert. 2) Check recent deploys (last 30 min). 3) If recent deploy – rollback immediately. 4) If not, check node status, etcd, control plane. 5) In parallel, inform team lead, start war room. 6) Status update to customer comms team.*

**Q2.** After the fix, in the RCA meeting, the manager asks – "Who is responsible?" What do you say?
> *Answer key: Blameless postmortem culture. Focus on process gap, not person. "There's a gap in our deployment process – staging doesn't have the same traffic pattern as prod. Let's add canary deploy and refine monitoring alerts." Professional maturity expected.*

**Q3.** A developer ran a DELETE query in the production DB. 50,000 records are gone. Backup is 6 hours old. What do you do?
> *Answer key: 1) Immediate: stop all writes to the table, snapshot current state. 2) PITR (Point-in-Time Recovery) – RDS/Aurora supports this. 3) Replay binary logs between backup and delete timestamp. 4) Restore to new DB, validate, switch. 5) Post-incident: revoke access, audit logs, 4-eye principle for prod queries.*

**Q4.** The CEO calls – "Competitors deploy daily. We deploy monthly. Fix it in 2 weeks." How do you start?
> *Answer key: Assess current pipeline bottlenecks – manual tests, approvals, deploy windows. Identify automation gaps. Propose trunk-based + feature flags. Staged rollout: daily to dev → weekly to stg → biweekly to prod (phase 1), then daily prod. Negotiate realistic timeline.*

**Q5.** You ran `terraform apply`. Production VPC was accidentally deleted. 200 services are down. What do you do in the next 5 minutes?
> *Answer key: 1) Don't run anything. Preserve state. 2) Declare incident – sev1. 3) Inspect `terraform state` – what got destroyed. 4) Recovery: last known good state file, re-apply. 5) If unique resource IDs are lost – DNS/IP changes – update dependent services. 6) Learning: `lifecycle prevent_destroy`, separate state per critical module.*

**Q6.** A crypto miner is running inside a pod. Security team alerted. What do you do first?
> *Answer key: 1) Isolate – network policy deny all, or cordon node. 2) Don't delete the pod – needed for forensics. 3) Take memory dump, process list. 4) Check image source – compromised registry? 5) Rotate all secrets the pod accessed. 6) Audit RBAC – how did the attacker deploy? 7) Image signing + admission controller (Kyverno/OPA).*

**Q7.** Your manager says – "Kubernetes is complicated. Let's migrate to Docker Swarm." What's your response?
> *Answer key: Professional disagreement. Argue with data: K8s ecosystem, skills availability, feature parity gap, migration cost, vendor support. Offer a POC comparison. But if it's a business decision, execute professionally. Shows maturity.*

---

## 9. Linux, Scripting & Troubleshooting

**Q1.** Server CPU is at 100%. How do you debug without killing the process to identify it?
> *Answer key: `top`/`htop` → PID, `ps -ef`, `strace -p PID`, `/proc/PID/status`, `lsof -p PID`, `iotop` for IO. If it's an app – thread dump (`jstack` for Java). `perf top` for system-wide.*

**Q2.** Disk is 100% full. `df -h` shows it, but `du -sh *` shows no space used. Reason?
> *Answer key: Deleted file held open by a process. `lsof | grep deleted`. Restart process or `> /proc/PID/fd/N` to truncate. Hidden mount points (df shows mounts, du doesn't cross them).*

**Q3.** Write a Python script to delete files older than 7 days in a directory.
> *Answer key: `os.walk`, `os.path.getmtime`, `time.time() - 7*86400`, `os.remove`. Or: `find /path -type f -mtime +7 -delete` (simpler).*

**Q4.** How do you run the same command in parallel on 100 servers in Bash?
> *Answer key: GNU parallel, `xargs -P`, Ansible ad-hoc, pdsh, or `for` loop with `&`. Best: Ansible for idempotency and logging.*

**Q5.** A log file is 50GB. How do you count how many "ERROR" occurred in the last 1 hour?
> *Answer key: `awk` with date filter, or `tail -n 100000 log | grep -c ERROR`. Better: `journalctl --since '1 hour ago' | grep -c ERROR`, or ELK/Grafana Loki.*

---

## 10. Behavioral & Closing Questions

**Q1.** What is the biggest mistake you made in 8 years? What did you learn?
> *Answer key: Expect an honest answer. Blame-shifting is a red flag. Specific incident + learning + process change expected.*

**Q2.** A team member isn't listening to what you say. How do you handle it?
> *Answer key: Data-driven discussion, 1-on-1, escalate only after attempts, focus on common goal not ego.*

**Q3.** How do you handle burnout in on-call rotation?
> *Answer key: Mature answer – automate runbooks, tune alerts (reduce noise), fair rotation, raise with manager. Not "I don't get burnout."*

**Q4.** What is your career goal for the next 3 years?
> *Answer key: Clear direction – Tech Lead / Architect / Senior SRE. "Same role" shows no growth mindset.*

**Q5.** Do you have any questions for us?
> *Answer key: Red flag if candidate has no questions. Expect questions like team structure, on-call load, future of tech stack, growth path.*

---

## 11. Evaluation Sheet – Scoring Template

> *Rate each section 1–5. Below 3 is a concern.*

- Kubernetes depth: ___ / 5
- Docker & containerization: ___ / 5
- AWS / Cloud: ___ / 5
- Terraform / IaC: ___ / 5
- CI/CD (Jenkins, ArgoCD, Helm): ___ / 5
- Real-time troubleshooting: ___ / 5
- Scripting (Python/Bash): ___ / 5
- Communication clarity: ___ / 5
- Behavioral maturity: ___ / 5
- Leadership (expected for 8 yrs): ___ / 5

**Total: ___ / 50**

**Recommendation:** [ ] Strong Hire &nbsp; [ ] Hire &nbsp; [ ] No Hire &nbsp; [ ] Strong No Hire

---

## Final Tips for Interviewer

- Make candidate comfortable in first 5 mins – offer water, small talk
- Ask for real numbers – "how many pods", "team size", "downtime minutes"
- Use "Why" questions a lot – "Why did you choose X over Y?"
- Go 3–4 layers deep on one topic. Depth matters more than breadth
- Take notes question-wise – don't rely on memory
