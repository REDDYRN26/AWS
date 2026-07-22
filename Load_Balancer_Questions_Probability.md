# AWS Load Balancer Questions - REAL Interview Probability

## DEFINITELY WILL ASK (95-100% Probability)

### Must Know - Basic Level

**1. "ELB vs ALB vs NLB difference? When would you use each?"**
- **Probability: 99%** - Almost every AWS interview asks this
- **Real answer they want:**
  - ELB = Layer 4+7, legacy (EC2-Classic), avoid in 2024
  - ALB = Layer 7, HTTP/HTTPS, path/hostname routing, microservices friendly
  - NLB = Layer 4, extreme performance, TCP/UDP, millions of concurrent connections
  - Rule of thumb: ALB default choice for web apps, NLB only if you need extreme performance

**2. "What's the difference between Layer 4 and Layer 7 load balancing?"**
- **Probability: 95%** - They ask in different ways
- **Real answer:** L4 sees IP:port, L7 sees HTTP headers/paths/hostnames. L7 smarter routing but slightly higher latency.

**3. "Path-based vs hostname-based routing. Give me a real example."**
- **Probability: 90%** - Almost always asked for ALB
- **Real example they like:**
  - Path-based: `example.com/api` → API service, `example.com/images` → image service
  - Hostname-based: `api.example.com` → API service, `web.example.com` → web service
  - **Pro tip to say:** Path-based simpler for same domain, hostname-based cleaner separation

**4. "What are target groups? Why are they useful?"**
- **Probability: 85%** - Follow-up to "how does ALB route?"
- **Real answer:** Decouple routing from backend. Same ALB can route to EC2, Lambda, IPs, on-prem servers. Enable blue-green deployments, multiple ports on same instance.

**5. "How does sticky sessions work in ALB? What problems can it cause?"**
- **Probability: 88%** - Very common real-world issue
- **Real answer:** Cookie-based session affinity. Client always routes to same backend.
  - **Problems:** Uneven load (some instances get 70%, others 30%), fails if instance dies, adds latency
  - **When to use:** Only if absolutely need session state on backend (prefer externalize to Redis)

---

## VERY LIKELY (70-95% Probability)

**6. "What happens during Blue-Green deployment with ALB?"**
- **Probability: 80%** - They want to know zero-downtime strategy
- **Answer:** Two target groups (Blue=current, Green=new). Deploy new code to Green. Switch listener rule to route to Green. Rollback by switching back. No downtime.

**7. "How do you implement SSL/TLS termination in ALB/NLB?"**
- **Probability: 75%** - Security-focused interviews always ask
- **Answer:** Upload cert to ACM, create HTTPS listener (port 443), backend can be HTTP. Reduces backend CPU. ALB can do SNI (multiple certs on same listener).

**8. "What is connection draining / deregistration delay? Why does it matter?"**
- **Probability: 82%** - Deployment/graceful shutdown questions trigger this
- **Answer:** After marking instance unhealthy, existing requests get max 300 seconds (configurable) to complete. New requests rejected. Critical for batch jobs, long-running requests.

**9. "Your backend is unhealthy but still getting traffic. Why?"**
- **Probability: 78%** - Troubleshooting scenario
- **Answer:** 
  - Health check not configured correctly (checking wrong port/path)
  - Threshold not reached yet (needs 2-3 failed checks to mark unhealthy)
  - Sticky sessions routing to specific unhealthy instance
  - Security group blocking health check

**10. "How would you debug why traffic isn't being distributed evenly?"**
- **Probability: 75%** - Real production issue
- **Answer:**
  - Check if sticky sessions enabled
  - Verify all instances passing health checks
  - Review CloudWatch RequestCount per instance
  - Use access logs to see actual distribution
  - Check instance capacity/load

---

## LIKELY (50-70% Probability)

**11. "Explain health checks - parameters you can configure."**
- **Probability: 65%** - Intermediate level
- **Answer:** Timeout (2-60s), interval (30s-5m), healthy threshold (2-10), unhealthy threshold (2-10), success code (200-399)

**12. "What's the difference between TCP vs HTTP health checks?"**
- **Probability: 60%** - ALB/NLB specific
- **Answer:** TCP = blind (just check port opens), HTTP = smart (can check response code and body). HTTP better for actual health validation.

**13. "How would you handle WebSocket connections with ALB?"**
- **Probability: 62%** - If company has real-time features
- **Answer:** Need sticky sessions because HTTP Upgrade, connection stays open. ALB treats as normal HTTP upgrade. NLB better for true persistent connections.

**14. "What are the costs differences between ALB and NLB?"**
- **Probability: 58%** - DevOps/architect interviews
- **Answer:** ALB charges per LB-hour + per GB. NLB cheaper per request but charges per GB data processed (can be expensive for high throughput). Cross-zone charges both.

**15. "What's the difference between cross-zone and zonal load balancing in NLB?"**
- **Probability: 55%** - Advanced ALB/NLB question
- **Answer:** Cross-zone = requests distributed across AZs (better resilience but costs more). Zonal = traffic stays in AZ (cheaper but uneven distribution).

---

## SOMEWHAT LIKELY (30-50% Probability)

**16. "How would you scale load balancer for millions of concurrent connections?"**
- **Probability: 45%** - Senior engineer or architecture role
- **Answer:** NLB auto-scales. Monitor ActiveFlow count. Set target tracking scaling policies. Use Route 53 for multi-LB.

**17. "Explain the three-way handshake in context of ALB/NLB."**
- **Probability: 35%** - Deep technical interviews
- **Answer:** ALB/NLB completes handshake with both client AND backend separately. Doubles latency slightly but allows inspection/modification.

**18. "What is the SurgeQueue metric in ELB? When would you see it?"**
- **Probability: 40%** - Legacy/troubleshooting focused
- **Answer:** Requests waiting in queue. When hits max 1000, new requests get 503 error. Indicates backends overwhelmed.

**19. "How would you implement active-active multi-region load balancing?"**
- **Probability: 42%** - Architecture interviews
- **Answer:** NLB in each region + Route 53 geolocation routing. Or use AWS Global Accelerator (anycast IP, automatic routing).

**20. "Can ALB route based on HTTP headers or query parameters?"**
- **Probability: 48%** - Advanced routing scenarios
- **Answer:** Yes, ALB can route on custom headers and query string. But doesn't offer full GraphQL/query param parsing - often need API Gateway instead.

---

## LESS LIKELY (10-30% Probability)

**21. "Explain 5-tuple hash in NLB."**
- **Probability: 25%** - Very technical, NLB-specific
- **Answer:** Source IP, source port, destination IP, destination port, protocol. Same flow always routes to same target.

**22. "How does NLB handle UDP traffic? Any gotchas?"**
- **Probability: 22%** - Not many use UDP
- **Answer:** NLB tracks UDP flows via timeout (120s default). If timeout expires, next packet might route elsewhere. App must handle UDP packet loss.

**23. "What is TIME_WAIT state and connection exhaustion?"**
- **Probability: 18%** - Very advanced networking
- **Answer:** TCP waits 60 seconds before closing port. If millions of connections, can exhaust ephemeral port range. Solution: connection pooling, increase port range.

**24. "Explain request deduplication and idempotency in context of LB."**
- **Probability: 20%** - Database/payment system interviews
- **Answer:** If request retried and first response lost, might duplicate process. Need idempotency tokens at app level to prevent double-charging payments.

**25. "How would you implement rate limiting at LB level?"**
- **Probability: 28%** - Security/API focused
- **Answer:** ALB/NLB don't have native rate limiting. Use WAF (Layer 7) or API Gateway. Implement in application layer if needed.

---

## RARELY ASKED (< 10% Probability)

**26. "Explain listener rule priority and rule conflicts."**
- **Probability: 8%** - Very specific edge case

**27. "How does ALB handle HTTP/2 multiplexing?"**
- **Probability: 5%** - Rarely comes up

**28. "Can you route to on-premises targets via IP target group?"**
- **Probability: 6%** - Hybrid-specific

**29. "Explain graceful shutdown with NLB vs ALB differences."**
- **Probability: 7%** - Very detailed

**30. "UDP hole punching and NAT traversal with NLB."**
- **Probability: 3%** - Only gaming/P2P companies

---

## WHAT INTERVIEWERS ACTUALLY CARE ABOUT

### Top 5 Things They're Testing:

1. **Do you know when to use each LB?** (ALB vs NLB decision-making)
2. **Can you troubleshoot a real problem?** (sticky sessions, health checks, uneven load)
3. **Do you understand the trade-offs?** (cost vs performance, simplicity vs capability)
4. **Can you design a zero-downtime deployment?** (blue-green, canary, rolling)
5. **Do you know Layer 4 vs Layer 7?** (what each can/can't do)

---

## INTERVIEW PREPARATION STRATEGY

### Must Master (Study 100%):
- [ ] ELB vs ALB vs NLB - when to use each
- [ ] Path vs hostname routing
- [ ] Sticky sessions + problems
- [ ] Health checks
- [ ] Blue-Green deployment with ALB
- [ ] SSL/TLS termination
- [ ] Connection draining / deregistration delay
- [ ] Target groups concept
- [ ] How to troubleshoot uneven load distribution

### Should Know (Study 80%):
- [ ] Cross-zone load balancing
- [ ] Cost differences
- [ ] WebSocket handling
- [ ] Security groups + health checks
- [ ] ALB access logs vs request tracing
- [ ] Scaling load balancers
- [ ] Multi-region setup

### Nice to Have (Study 30%):
- [ ] TCP vs UDP in NLB
- [ ] 5-tuple hash
- [ ] REQUEST deduplication
- [ ] TIME_WAIT state
- [ ] Advanced networking protocols

---

## REAL INTERVIEW FLOW (Typical)

**Interviewer:** "Design a load balancing architecture for a microservices app."

**Expected Response:**
1. Ask clarification questions (throughput? protocols? regions?)
2. Recommend ALB for HTTP microservices
3. Explain path-based routing for different services
4. Mention zero-downtime deployment (blue-green)
5. Discuss security (security groups, SSL termination)
6. Ask about cost constraints

**Interviewer Satisfaction Level:** ⭐⭐⭐⭐⭐

**Then they dig deeper:**
- "But what if we have WebSocket? Still ALB?"
- "How do you handle sticky sessions without load imbalance?"
- "What if traffic is 10 million requests/second?"
- "How do you debug if 30% of traffic goes to one instance?"

**Your answer depth matters more than knowing all 30 questions.**

---

## REAL SCENARIOS THAT DEFINITELY COME UP

**"We deployed new code and traffic spiked to one backend instance. Others are idle. Why?"**
- Answer shows: sticky sessions knowledge, debugging skills, understanding of hash distribution

**"Health checks pass but users say service is slow."**
- Answer shows: troubleshooting approach, understanding of what health checks do/don't validate

**"We need zero-downtime deployment. How would you do it with ALB?"**
- Answer shows: practical ops knowledge, understanding of deregistration delay, target group switching

**"One client is causing 1000 requests/second but legit traffic is normal. How to block?"**
- Answer shows: understanding of rate limiting (WAF), security posture, ALB limitations

**"We're paying $500/month for NLB but using only 10% capacity. How to optimize?"**
- Answer shows: cost awareness, understanding of ALB vs NLB trade-offs

---

## INTERVIEW RED FLAGS (Things That Hurt You)

❌ "I always use ALB for everything" (doesn't understand NLB exists or when to use)
❌ "Health checks are just checking if port is open" (doesn't understand HTTP checks)
❌ "Sticky sessions solve all performance issues" (doesn't know the problems)
❌ "I've never had to troubleshoot LB issues" (sounds inexperienced)
❌ "I don't know the difference between Layer 4 and 7" (fundamental gap)
❌ "ELB, ALB, NLB are all basically the same" (wrong)

---

## PROBABILITY CHEAT SHEET

| Question | Probability | Study Priority |
|----------|------------|-----------------|
| ELB vs ALB vs NLB? | 99% | 🔴 MUST |
| Layer 4 vs Layer 7? | 95% | 🔴 MUST |
| Path vs hostname routing? | 90% | 🔴 MUST |
| What is target group? | 85% | 🔴 MUST |
| Sticky sessions problems? | 88% | 🔴 MUST |
| Blue-Green deployment? | 80% | 🔴 MUST |
| SSL termination? | 75% | 🟠 SHOULD |
| Connection draining? | 82% | 🔴 MUST |
| Troubleshoot uneven load? | 75% | 🟠 SHOULD |
| Health check parameters? | 65% | 🟠 SHOULD |
| Cross-zone LB? | 55% | 🟡 NICE |
| WebSocket + ALB? | 62% | 🟠 SHOULD |
| Cost optimization? | 58% | 🟡 NICE |
| Multi-region scaling? | 45% | 🟡 NICE |
| 5-tuple hash NLB? | 25% | ⚪ SKIP |
| UDP handling? | 22% | ⚪ SKIP |
| TIME_WAIT state? | 18% | ⚪ SKIP |

