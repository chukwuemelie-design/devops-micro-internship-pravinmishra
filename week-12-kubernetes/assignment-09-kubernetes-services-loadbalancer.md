# Assignment 09 — Kubernetes Services: LoadBalancer on AKS

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Deploy a healthy NGINX application to Azure Kubernetes Service (AKS), expose it publicly using a LoadBalancer Service, observe readiness behavior, and scale the application behind the same public endpoint.

---

# Task 1 — Create or Connect to an AKS Cluster

## Goal

Create a small AKS cluster for this lab, or connect to an existing approved AKS lab cluster.

### Evidence

#### Screenshot 1 — Output of `kubectl get nodes` showing ready AKS node(s)

<img width="909" height="148" alt="Ass9-ss1" src="https://github.com/user-attachments/assets/e4d50e51-43aa-4d88-baf6-3dcd396a846f" />


### Notes

**1. Why does this lab use a cloud-capable Kubernetes environment such as AKS for the LoadBalancer Service?**

AKS integrates with Azure networking and load-balancing services. This allows Kubernetes to provision an external load balancer and public IP address for exposing applications to the internet.

**2. What Azure resources can Kubernetes provision when a LoadBalancer Service is created?**

Depending on the AKS networking configuration, Kubernetes can provision or configure an Azure Load Balancer, a public IP address, and the associated load-balancing rules and backend pool configuration.

**3. Why is it important to delete unused AKS resources after the lab?**

Unused AKS resources can continue generating Azure charges. Deleting resources that are no longer needed helps control costs, avoid unnecessary resource consumption, and maintain a clean cloud environment.

---

# Task 2 — Set Up Your Lab Directory

## Goal

Create and enter a dedicated directory for the LoadBalancer Service manifests.

### Evidence

#### Screenshot 2 — Terminal showing the output of `pwd` confirming that you are working inside `~/k8s-labs/services/loadbalancer`

<img width="502" height="118" alt="Ass9-ss2" src="https://github.com/user-attachments/assets/5e55720d-bcba-4351-8ec3-018382ce29e2" />


### Notes

**1. Why is it useful to keep Kubernetes manifests organized in dedicated directories?**

Dedicated directories make project files easier to manage, maintain, locate, troubleshoot, and reuse. They also help keep different Kubernetes assignments separate.

**2. What are the names of the main Deployment and LoadBalancer Service YAML files you will create in this assignment?**

Deployment: 00-nginx-deploy.yaml



LoadBalancer Service: 01-nginx-svc-loadbalancer.yaml

---

# Task 3 — Deploy a Healthy NGINX Application

## Goal

Create a healthy NGINX Deployment with readiness and liveness probes.

### Evidence

#### Screenshot 3 — Completed `00-nginx-deploy.yaml` manifest

<img width="643" height="388" alt="Ass9-ss3" src="https://github.com/user-attachments/assets/880f1fab-acfe-46bb-af9a-e9382795ea2b" />


---

#### Screenshot 4 — Output showing successful rollout and two ready NGINX Pods

<img width="874" height="143" alt="Ass9-ss4" src="https://github.com/user-attachments/assets/7639a0cd-fea9-4387-8857-caedb0766ebc" />


### Notes

**1. Why must Pods be ready before they should receive public traffic?**

Pods must be ready to ensure they can successfully handle incoming requests. Sending traffic to unready Pods could result in failed requests or poor application availability.

**2. What is the role of the readiness probe in this Deployment?**

The readiness probe checks whether the NGINX container is ready to accept requests. Kubernetes uses its result to determine whether the Pod should receive Service traffic.

**3. What is the role of the liveness probe in this Deployment?**

The liveness probe checks whether the NGINX container is still healthy and responsive. If it repeatedly fails, Kubernetes restarts the unhealthy container.

---

# Task 4 — Create and Test the LoadBalancer Service

## Goal

Create a public LoadBalancer Service and verify that the NGINX application is reachable from the internet.

### Evidence

#### Screenshot 5 — Completed `01-nginx-svc-loadbalancer.yaml` manifest

<img width="746" height="148" alt="Ass9-ss5" src="https://github.com/user-attachments/assets/a911db38-ce75-4f9c-935b-bbfe70941bf3" />


---

#### Screenshot 6 — Output showing the Service with an assigned EXTERNAL-IP

<img width="854" height="152" alt="Ass9-ss6" src="https://github.com/user-attachments/assets/413dff18-3ff5-4ccd-a397-0de9bef019b5" />


---

#### Screenshot 7 — Successful curl output from the public IP

<img width="899" height="133" alt="Ass9-ss7" src="https://github.com/user-attachments/assets/f733892c-ea80-41bf-bf95-e3100b771a16" />


### Notes

**1. What does `type: LoadBalancer` instruct AKS to do?**

It instructs Kubernetes to expose the Service through an external load balancer. In AKS, this integrates with Azure networking to provide an externally accessible IP address.

**2. What public IP was assigned to your Service?**

Enter the actual public IP displayed in your Service output under EXTERNAL-IP.

**3. How does the public request eventually reach the NGINX Pods?**

A client sends a request to the public IP. The Azure Load Balancer forwards the request to the Kubernetes Service, which routes it to one of the available, ready NGINX Pods through its endpoints.

**4. Why should you wait for EXTERNAL-IP before testing the Service?**

The external IP may initially show pending while Azure provisions and configures the load balancer. Waiting ensures the public endpoint is assigned and ready for testing.

---

# Task 5 — Observe How Readiness Protects Public Traffic

## Goal

Intentionally break readiness and observe how Kubernetes excludes unready Pods from ready Service backends while existing healthy Pods continue serving public traffic.

### Evidence

#### Screenshot 8 — Completed `02-readiness-broken.yaml` patch file

<img width="643" height="239" alt="Ass9-ss8" src="https://github.com/user-attachments/assets/9fde23fe-11a6-4f31-a578-28451143cdf1" />


---

#### Screenshot 9 — Output showing the expected incomplete rollout or readiness failure

<img width="797" height="139" alt="Ass9-ss9" src="https://github.com/user-attachments/assets/8f49c307-a545-430b-a1cf-38e01f2eb1ae" />


---

#### Screenshot 10 — EndpointSlice output showing ready healthy backends and the effect of the readiness failure

<img width="709" height="386" alt="Ass9-ss10" src="https://github.com/user-attachments/assets/091455bb-a1ad-4a67-ad65-8b9c0bb7ce51" />


---

#### Screenshot 11 — Successful public curl response after recovery

<img width="794" height="222" alt="Ass9-ss11" src="https://github.com/user-attachments/assets/4c9df71b-a449-4463-a3b6-b820ec0cf2da" />


### Notes

**1. Why did the rollout not complete?**

The rollout did not complete because the new Pods failed their readiness probes. Kubernetes could not consider them available, so it waited instead of completing the replacement process.

**2. Why can the public endpoint still work while new Pods fail readiness?**

The existing healthy Pods remain available as Service backends. The LoadBalancer continues forwarding requests to those ready Pods while the new Pods are excluded.

**3. What would happen if Kubernetes routed traffic to Pods before they were ready?**

Users could experience failed requests, connection errors, or incomplete responses because the application might not yet be able to handle incoming traffic.

**4. How did you restore the Deployment successfully?**

I restored the correct readiness probe configuration and applied the updated Deployment manifest. The new Pods then passed their readiness checks, became available, and allowed the rollout to complete successfully.

---

# Task 6 — Scale the Deployment Behind the Load Balancer

## Goal

Scale the NGINX Deployment and confirm that more ready Pods become available behind the same public endpoint.

### Evidence

#### Screenshot 12 — Two ready NGINX Pods verified initially; four ready Pods verified after scaling

<img width="935" height="244" alt="Ass9-ss12" src="https://github.com/user-attachments/assets/fb046a22-acdf-4e7b-8466-4aaeb15744a1" />


---

#### Screenshot 13 — Successful public curl response after scaling

<img width="934" height="221" alt="Ass9-ss13" src="https://github.com/user-attachments/assets/f92d977d-16ae-41b5-a651-18b83befe70b" />


### Notes

**1. What changed after scaling the Deployment from two to four replicas?**

The Deployment increased its desired replica count from two to four, creating two additional NGINX Pods. After they became ready, four Pods were available to handle application traffic.

**2. Did the public IP change after scaling? Explain why or why not.**

No. The public IP remained the same because scaling changed the number of backend Pods, not the LoadBalancer Service configuration or its external IP address.

**3. Why does scaling behind a LoadBalancer Service not require users to change the URL or IP they use?**

The LoadBalancer Service provides a stable public endpoint. Kubernetes and Azure update the backend routing configuration as Pods are added or removed, so users continue accessing the application through the same IP address.

---

# Task 7 — Share Your LoadBalancer Lab on LinkedIn

## Goal

Share what you learned about exposing an NGINX application publicly through a LoadBalancer Service on AKS.

### Evidence

#### Screenshot 14 — Published LinkedIn post about your LoadBalancer lab on AKS



#### LinkedIn post link



---

# Submission Instructions

- Include the completed YAML files:
  - `00-nginx-deploy.yaml`
  - `01-nginx-svc-loadbalancer.yaml`
  - `02-readiness-broken.yaml`
- Add all required screenshots, numbered 1–14, under their corresponding Evidence sections.
- Use clear, readable screenshots showing the relevant commands, configuration, and outputs.
- Ensure your full name is visible in the required screenshots.
- Complete the Notes for Tasks 1–6 in your own words.
- Capture all required evidence before cleaning up Azure resources.
- Plan or complete Azure cleanup according to your lab ownership and reuse arrangements.
- Do not expose passwords, tokens, private keys, cluster credentials, kubeconfig contents, subscription IDs, account IDs, or infrastructure details you are not permitted to share.

---

# Completion Checklist

- [ ] Created or connected to an approved AKS lab cluster
- [ ] Confirmed ready AKS node(s) (Screenshot 1)
- [ ] Created and entered the LoadBalancer lab directory (Screenshot 2)
- [ ] Created the NGINX Deployment manifest (Screenshot 3)
- [ ] Verified successful rollout and two ready NGINX Pods initially (Screenshot 4)
- [ ] Created the LoadBalancer Service manifest (Screenshot 5)
- [ ] Verified the assigned external IP (Screenshot 6)
- [ ] Verified successful public access using curl (Screenshot 7)
- [ ] Created the broken readiness patch (Screenshot 8)
- [ ] Observed the expected incomplete rollout or readiness failure (Screenshot 9)
- [ ] Inspected EndpointSlices during the readiness failure (Screenshot 10)
- [ ] Restored the healthy Deployment and verified public access (Screenshot 11)
- [ ] Scaled the Deployment to four ready Pods and verified updated Service backends (Screenshot 12)
- [ ] Verified public access after scaling (Screenshot 13)
- [ ] Completed the Notes for Tasks 1–6
- [ ] Published the LinkedIn post (Screenshot 14)
- [ ] Included all required YAML files and screenshots
- [ ] Planned or completed Azure cleanup
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
