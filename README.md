# SPLUNK POWER USER COMPLETE GUIDE - README

## 📚 Overview

This package contains **3 comprehensive guides** to help you become a **Splunk Power User** for Linux System Administration and DevOps Infrastructure monitoring.

**Your Background:** 7 years IT experience with:
- 4 years Linux Administration + NetApp Storage
- Kubernetes & OpenShift
- AWS ROSA
- Terraform
- Git, Jenkins, ArgoCD
- Helm, Docker

These guides are specifically tailored for your DevOps background.

---

## 📦 What's Included

### File 1: `50_best_queries_english.md`
**The Most Comprehensive Query Reference**

**Contains:**
- ✅ 50 production-ready Splunk queries
- ✅ 5 bonus dashboard queries
- ✅ Total: 55 queries

**Organized Into 6 Sections:**

1. **User Authentication & Access Control** (8 queries)
   - Failed login detection
   - Sudo command auditing
   - Root user monitoring
   - Privilege escalation detection

2. **System Errors & Critical Failures** (10 queries)
   - Kernel panic detection
   - Out of memory alerts
   - Service crashes
   - Application segmentation faults

3. **Performance & Resource Monitoring** (9 queries)
   - CPU usage detection
   - Memory consumption alerts
   - Disk space monitoring
   - Load average tracking

4. **Network Monitoring & Connectivity** (8 queries)
   - Connection refused errors
   - Port scanning detection
   - DNS resolution failures
   - Firewall blocks

5. **Filesystem & Storage Monitoring** (7 queries)
   - File permission changes
   - File deletion auditing
   - Mount/unmount events
   - Inode exhaustion

6. **Security & Compliance Auditing** (8 queries)
   - Privilege escalation attempts
   - Firewall rule changes
   - Audit log tampering
   - Security threat detection

**Each Query Includes:**
- Purpose: Why use this query
- When to use: Practical guidance
- Copy-paste ready code

**Best For:** Detailed learning and understanding each query deeply

---

### File 2: `splunk_quick_reference_english.md`
**The Daily Go-To Quick Reference**

**Contains:**
- ✅ 55+ quick copy-paste queries
- ✅ Minimal explanation
- ✅ Maximum usability

**Organized Into 9 Quick-Reference Sections:**

1. **Critical/Urgent Queries** (5 queries)
   - For immediate crisis response
   - Errors, failed logins, disk full, memory, crashes

2. **System Health** (5 queries)
   - Error trends, reboots, CPU, memory

3. **User & Access** (8 queries)
   - Login tracking, sudo auditing, SSH keys

4. **Services & Processes** (7 queries)
   - Service restarts, crashes, systemd, cron

5. **Network** (7 queries)
   - Connections, DNS, firewall, SSH

6. **Storage & Files** (7 queries)
   - Disk errors, permissions, files, mount points

7. **Security** (6 queries)
   - Privilege escalation, unauthorized access, firewalls

8. **Performance & Load** (5 queries)
   - Load average, memory pressure, limits

9. **Dashboard Queries** (5 queries)
   - For creating dashboards

**Additional Sections:**
- Quick tips (10 optimization tips)
- Common field names
- When to use each query
- Troubleshooting guide
- Save/Alert/Schedule instructions

**Best For:** Day-to-day usage, quick lookups, rapid copy-paste

---

### File 3: `splunk_for_devops_english.md`
**Integration with Your DevOps Stack**

**Contains:**
- ✅ Queries for all your tools
- ✅ Setup guides for each platform
- ✅ Combined monitoring dashboards

**10 Major Sections:**

1. **Kubernetes Cluster Monitoring** (7 queries)
   - Pod health checks
   - Crash detection
   - Memory usage alerts
   - Restart spike alerts
   - Pending pods
   - Node resource pressure

2. **OpenShift Container Platform** (8 queries)
   - Failed deployments
   - Node status
   - Image pull failures
   - Operator health
   - PVC issues
   - Route traffic
   - Etcd cluster health

3. **AWS / ROSA Monitoring** (9 queries)
   - IAM permission changes
   - EC2 instance state changes
   - ROSA cluster API calls
   - Failed API calls
   - S3 bucket access
   - Root account activity
   - VPC flow logs

4. **Jenkins CI/CD Monitoring** (7 queries)
   - Build success rate
   - Failed builds alert
   - Long running builds
   - Pipeline stage failures
   - Build timeouts
   - Agent offline alerts
   - Test failures trend

5. **Docker Container Monitoring** (4 queries)
   - Container exits
   - Build failures
   - Network issues
   - Volume issues

6. **Terraform & IaC** (4 queries)
   - Plan failures
   - Apply failures
   - Infrastructure changes audit
   - Resource creation tracking

7. **Helm Chart Deployments** (3 queries)
   - Release failures
   - Release status
   - Rollback tracking

8. **ArgoCD Deployment Tracking** (3 queries)
   - Sync failures
   - Out of sync apps
   - Deployment errors

9. **Combined DevOps Dashboards** (3 queries)
   - Infrastructure health summary
   - All failed deployments
   - All alerts across stack

10. **Performance Dashboards** (3 queries)
    - Load trends
    - Success rate
    - Resource consumption

**Additional Content:**
- Step-by-step setup guide
- Useful commands for each tool
- Weekly roadmap for implementation
- DevOps-specific tips

**Best For:** Implementing Splunk across your entire infrastructure

---

## 🎯 How to Use These Files

### Quick Start (Today - 15 minutes)

**Step 1:** Open `splunk_quick_reference_english.md`

**Step 2:** Find Query #1 (Show All Errors Last 24 Hours)
```spl
index=main sourcetype=syslog (ERROR OR CRITICAL) earliest=-24h | stats count by host
```

**Step 3:** Log into your Splunk instance

**Step 4:** Click Search & Reporting

**Step 5:** Paste the query in the search bar

**Step 6:** Press Enter

**Step 7:** See your results!

---

### Week 1: Learning Phase
- **Use:** `50_best_queries_english.md`
- **Activity:** Read and understand 10 queries
- **Goal:** Learn SPL syntax and logic
- **Outcome:** Understand how queries work

---

### Week 2: Practice Phase
- **Use:** `splunk_quick_reference_english.md`
- **Activity:** Try 15-20 different queries
- **Goal:** Get comfortable with copy-paste
- **Outcome:** Start creating your own queries

---

### Week 3: Integration Phase
- **Use:** `splunk_for_devops_english.md`
- **Activity:** Set up Splunk connections to your tools
- **Goal:** Get your Kubernetes, Jenkins, AWS data into Splunk
- **Outcome:** See real infrastructure data

---

### Week 4+: Automation Phase
- **Use:** All 3 files as reference
- **Activity:** Create dashboards, set up alerts
- **Goal:** Automate monitoring
- **Outcome:** Production-ready dashboards

---

## 📊 Which File to Use When

| Situation | File to Use | Reason |
|-----------|------------|--------|
| Need quick answer NOW | Quick Reference | Fast, minimal explanation |
| Learning SPL syntax | 50 Best Queries | Detailed explanations |
| Setting up Kubernetes monitoring | DevOps for Splunk | Tool-specific setup |
| Creating dashboard | Quick Reference + 50 Best | Get ideas and syntax |
| Debugging slow query | 50 Best Queries | Has optimization tips |
| Setting up alerts | Quick Reference | Has alert instructions |
| Troubleshooting infrastructure | DevOps for Splunk | Has troubleshooting |

---

## 🔥 Top 10 Most Useful Queries

These are the queries you'll use most frequently:

### From Quick Reference:
1. **Show All Errors** - Daily check
2. **Failed Login Attempts** - Security
3. **Disk Almost Full** - Storage
4. **High CPU/Memory** - Performance
5. **Service Crashed** - Incident response

### From 50 Best:
6. **Kubernetes Pod Health** - DevOps
7. **Jenkins Build Success Rate** - CI/CD
8. **Failed AWS API Calls** - Cloud
9. **Privilege Escalation Attempts** - Security
10. **Infrastructure Load Trend** - Dashboard

---

## 💡 Pro Tips for Success

### Tip 1: Bookmark Quick Reference
Keep `splunk_quick_reference_english.md` bookmarked in your browser for instant access.

### Tip 2: Start with 5-Second Queries
Don't run complex queries first. Start with:
```spl
index=main | head 100
```

### Tip 3: Always Add Time Range
Faster searches:
```spl
index=main earliest=-24h
```
Much faster than:
```spl
index=main
```

### Tip 4: Test Before Making Alert
Run query manually first. Then save as alert.

### Tip 5: Use Exact Sourcetype
Instead of:
```spl
index=main ERROR
```
Use:
```spl
index=main sourcetype=syslog ERROR
```

### Tip 6: Group Results with Stats
Instead of:
```spl
index=main ERROR | head 1000
```
Use:
```spl
index=main ERROR | stats count by host
```

### Tip 7: Create Saved Searches
After running a query:
1. Click "Save As"
2. Select "Search"
3. Give it a name
4. Now it's reusable

### Tip 8: Schedule Reports
After saving search:
1. Click "Schedule"
2. Set daily frequency
3. Add email
4. Get daily reports automatically

---

## 📈 Expected Learning Timeline

**Day 1-2:** Understand basic SPL syntax
- Read Quick Reference
- Try 5-10 queries
- Understand `|` (pipe) concept

**Day 3-5:** Comfortable with queries
- Try all Quick Reference queries
- Understand `stats`, `where`, `table`
- Create first saved search

**Week 2:** Learning curves
- Read 50 Best Queries details
- Understand optimization
- Create first dashboard

**Week 3:** DevOps integration
- Set up Kubernetes monitoring
- Set up Jenkins monitoring
- Create combined dashboard

**Week 4+:** Production ready
- Set up critical alerts
- Automate daily reports
- Optimize searches

---

## 🎓 Next Steps After This Package

### Step 1: Master These Queries (2-4 weeks)
Complete all queries in this package.

### Step 2: Get Certified
- Take Splunk Core Certified Power User exam
- Takes 1-2 months of study

### Step 3: Advanced Skills
- Learn Advanced Search & Reporting
- Learn Splunk Dashboards and Visualizations
- Learn Splunk Administration

### Step 4: Contribute to Organization
- Implement Splunk in your company
- Train your team
- Create organization-specific dashboards

---

## ❓ Frequently Asked Questions

### Q1: Which file should I start with?
**A:** Start with `splunk_quick_reference_english.md`. It's quick and practical.

### Q2: Why are some queries different between files?
**A:** Quick Reference has simpler versions for speed. 50 Best Queries has detailed versions for learning.

### Q3: Can I use these queries on my production data?
**A:** YES! These are production-safe queries. They only READ data, never modify.

### Q4: How often should I review these queries?
**A:** 
- Week 1: Daily
- Week 2: Few times a week
- Week 3+: As needed for reference

### Q5: Can I modify these queries?
**A:** YES! Modify them for your specific needs:
- Change `host=*` to specific servers
- Change `earliest=-24h` to different time ranges
- Add/remove conditions

### Q6: How do I create an alert from these queries?
**A:** 
1. Run query
2. Click "Save As"
3. Choose "Alert"
4. Set trigger: "When results > 0"
5. Add action: Email/Slack/PagerDuty

### Q7: What if my logs aren't in Splunk yet?
**A:** Check `splunk_for_devops_english.md` for setup guides:
- Kubernetes: Use Helm install
- Jenkins: Configure log forwarding
- AWS: Enable CloudTrail
- Docker: Use Splunk Universal Forwarder

### Q8: Are there more queries available?
**A:** YES! These are just the most useful ones. Once you master these, explore Splunk docs for advanced queries.

### Q9: Can I use these for compliance/auditing?
**A:** YES! Many queries are specifically for compliance:
- User access auditing
- Privilege escalation detection
- File modification tracking
- System change auditing

### Q10: How do I optimize slow queries?
**A:** From `50_best_queries_english.md`:
- Add time range: `earliest=-24h`
- Use field search: `host=server` not `search host=server`
- Add `head 100` for testing
- Use `where` AFTER `stats`, not before

---

## 🚀 Getting Help

### If Splunk returns no results:
1. Check time range
2. Check sourcetype: `index=main | stats count by sourcetype`
3. Check if data is being collected
4. Simplify query and test step by step

### If query is too slow:
1. Add `earliest=-24h`
2. Use specific `host`
3. Add `head 100`
4. Use field search instead of regex

### If you can't find a specific query:
1. Use Ctrl+F to search within files
2. Check "When to Use" tables
3. Modify existing query to fit your need

---

## 📝 Your Action Items

### This Week:
- [ ] Download all 3 files
- [ ] Open Quick Reference
- [ ] Try Query #1 in your Splunk
- [ ] Try Query #2, #3, etc.

### Next Week:
- [ ] Read 50 Best Queries overview
- [ ] Pick 5 queries to implement
- [ ] Create first saved search

### Following Week:
- [ ] Review DevOps for Splunk
- [ ] Set up Kubernetes monitoring
- [ ] Create first dashboard

### Week 4:
- [ ] Create critical alerts
- [ ] Schedule daily reports
- [ ] Train your team

---

## 📄 File Summary

| File | Best For | Size | Read Time |
|------|----------|------|-----------|
| Quick Reference | Daily use | Short | 5-10 min |
| 50 Best Queries | Learning | Medium | 30-45 min |
| DevOps for Splunk | Infrastructure | Medium | 20-30 min |

---

## ✅ What You'll Be Able To Do

After completing all 3 files, you'll be able to:

- ✅ Write SPL queries from scratch
- ✅ Monitor your Linux servers
- ✅ Track user access and security
- ✅ Monitor your Kubernetes clusters
- ✅ Monitor your Jenkins pipelines
- ✅ Track AWS/ROSA infrastructure
- ✅ Create professional dashboards
- ✅ Set up production alerts
- ✅ Schedule automated reports
- ✅ Troubleshoot infrastructure issues
- ✅ Audit system changes
- ✅ Become a Splunk Power User

---

## 🎯 Your Splunk Power User Journey

```
START HERE
    ↓
Quick Reference (5-10 queries)
    ↓
Understand SPL basics
    ↓
50 Best Queries (deep learning)
    ↓
Advanced queries & optimization
    ↓
DevOps for Splunk (integration)
    ↓
Create dashboards
    ↓
Set up alerts
    ↓
Schedule reports
    ↓
SPLUNK POWER USER! 🚀
```

---

## 📞 Support & Resources

### Official Splunk Documentation
- https://docs.splunk.com
- https://www.splunk.com/en_us/training.html

### Splunk Community
- https://community.splunk.com
- Thousands of experts ready to help

### Learning Paths
- Splunk Core Certified Power User
- Splunk Core Certified Advanced Power User
- Splunk Certified Administrator

---

## 📦 Package Contents Checklist

- ✅ `50_best_queries_english.md` (55 detailed queries)
- ✅ `splunk_quick_reference_english.md` (55 quick queries)
- ✅ `splunk_for_devops_english.md` (DevOps integration guide)
- ✅ `README.md` (This file)

---

## 🎉 Congratulations!

You now have everything you need to become a **Splunk Power User**!

Start with the Quick Reference today. In 4 weeks, you'll be monitoring your entire infrastructure from Splunk.

**Happy Splunking! 🚀**

---

**Last Updated:** April 2026
**Version:** 1.0
**For:** DevOps Engineers with Linux, Kubernetes, OpenShift, AWS, Jenkins experience
