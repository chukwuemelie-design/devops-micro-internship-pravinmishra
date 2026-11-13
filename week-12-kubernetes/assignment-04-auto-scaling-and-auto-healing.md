# Assignment 4 — Auto-Scaling & Auto-Healing: Let Kubernetes React for You

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Create Deployments, test Pod auto-healing, configure CPU requests and limits, verify Metrics Server, and create a Horizontal Pod Autoscaler.

Optionally generate CPU load to observe how the HPA responds.

---

# Task 0 — Check the Existing Cluster Resources

## Goal

Confirm that you are connected to your learning cluster and inspect existing Deployments and HPAs.

### Evidence

No screenshots required.

---

# Task 1 — Set Up Your Lab Directory

## Goal

Create and enter a dedicated directory for the auto-scaling lab.

### Evidence

#### Screenshot 1 — Terminal showing that you are working inside `~/k8s-labs/autoscaling`

<img width="383" height="96" alt="Ass4-ss1" src="https://github.com/user-attachments/assets/6c6d3c7a-776d-485f-8e14-3d1a7eac7d65" />


---

### Notes

**1. What is the difference between auto-healing and auto-scaling?**

Auto-healing automatically replaces failed or deleted Pods to maintain the desired number of replicas. Auto-scaling automatically increases or decreases the number of Pods based on resource usage or application demand.

---

**2. Which Kubernetes feature will you use for CPU-based auto-scaling?**

Horizontal Pod Autoscaler (HPA).

---

# Task 2 — Test Auto-Healing with a Deployment

## Goal

Create a Deployment with two replicas, delete one Pod, and observe Kubernetes create a replacement Pod.

### Evidence

#### Screenshot 2 — Completed `autohealing-deployment.yaml` manifest

<img width="572" height="100" alt="Ass4-ss2" src="https://github.com/user-attachments/assets/c149fae5-af7b-464d-911a-7ef73f0adb04" />


---

#### Screenshot 3 — Output of `kubectl get pods` showing two running Pods before deletion

<img width="527" height="103" alt="Ass4-ss3" src="https://github.com/user-attachments/assets/9dd2a4c2-febd-464f-8b17-ec57c3db17b8" />


---

#### Screenshot 4 — Output of the `kubectl delete pod` command

<img width="569" height="67" alt="Ass4-ss4" src="https://github.com/user-attachments/assets/194ad671-7c03-4a92-8436-081b98f24ba8" />


---

#### Screenshot 5 — Output of `kubectl get pods` showing the replacement Pod and two running Pods

<img width="493" height="109" alt="Ass4-ss5" src="https://github.com/user-attachments/assets/7325d952-73d8-49a1-a166-c1eb7901e3f2" />


---

### Notes

**1. What happened after you deleted one Pod?**

After I deleted one Pod, Kubernetes automatically created a replacement Pod to maintain the desired number of two replicas.

---

**2. What Kubernetes resource maintains the desired replica count for this Deployment?**

The ReplicaSet manages the desired replica count by ensuring that the specified number of Pods is maintained.

---

**3. What proves that auto-healing worked?**

The appearance of a replacement Pod with a different name and the return of the Deployment to two running and ready Pods prove that auto-healing worked successfully.

---

# Task 3 — Create a Deployment for HPA

## Goal

Create a Deployment with CPU resource requests and limits so that HPA can calculate CPU utilization.

### Evidence

#### Screenshot 6 — Completed `hpa-deployment.yaml` manifest showing CPU requests and limits

<img width="512" height="154" alt="Ass4-ss6" src="https://github.com/user-attachments/assets/7cac71ca-6350-4be5-905b-ddde1e367e90" />


---

#### Screenshot 7 — Output showing `nginx-deployment` and its running Pods

<img width="602" height="312" alt="Ass4-ss7" src="https://github.com/user-attachments/assets/ce4baab5-ece0-4df2-9798-b2374457491d" />


---

### Notes

**1. What does `cpu: "100m"` mean?**

It means the container requests 100 millicores of CPU, which is equivalent to 0.1 of one CPU core.

---

**2. What is the difference between a CPU request and a CPU limit?**

A CPU request specifies the amount of CPU resources Kubernetes reserves for a container when scheduling it. A CPU limit specifies the maximum CPU usage the container is allowed to consume.

---

**3. Why does CPU-based HPA require CPU requests?**

CPU-based HPA uses CPU requests as the baseline for calculating CPU utilization percentages. Without CPU requests, Kubernetes cannot accurately calculate the percentage of CPU utilization for scaling decisions.

---

# Task 4 — Enable and Verify Metrics Server

## Goal

Ensure the cluster can collect resource metrics for the HPA.

### Evidence

#### Screenshot 8 — Output showing Metrics Server running, or `kubectl top pods` displaying Pod metrics

<img width="514" height="149" alt="Ass4-ss8" src="https://github.com/user-attachments/assets/a1cf7bd4-fc10-4cb8-9ee7-8bdb0f7d4c8e" />


---

### Notes

**1. Why does HPA need Metrics Server?**

CPU-based HPA uses CPU requests as the baseline for calculating CPU utilization percentages. Without CPU requests, Kubernetes cannot accurately calculate the percentage of CPU utilization for scaling decisions.

---

**2. What issue can occur if Metrics Server is not working?**

HPA needs Metrics Server to collect CPU and memory usage metrics from Kubernetes Pods and provide the information required to make scaling decisions.

---

**3. What does `kubectl top pods` help you check?**

HPA may be unable to retrieve resource metrics, resulting in unknown CPU utilization and preventing automatic scaling decisions.

---

# Task 5 — Create a Horizontal Pod Autoscaler

## Goal

Create an HPA that keeps at least two Pods running and can scale up to five Pods when average CPU utilization exceeds 50%.

### Evidence

#### Screenshot 9 — The imperative command output OR completed `hpa-nginx.yaml` manifest

<img width="593" height="122" alt="Ass4-ss9" src="https://github.com/user-attachments/assets/4242d235-54a4-4556-8b46-779ce80859cd" />


---

#### Screenshot 10 — Output of `kubectl get hpa` and the relevant `kubectl describe hpa` command

<img width="612" height="107" alt="Ass4-ss10" src="https://github.com/user-attachments/assets/91e2431a-75db-46d9-a05a-94992e0202f0" />


---

### Notes

**1. What does `minReplicas: 2` mean?**

It specifies that the Deployment must maintain at least two Pods, even when resource utilization is low.

---

**2. What does `maxReplicas: 5` mean?**

It specifies that the HPA can increase the number of Pods up to a maximum of five when resource utilization increases.

---

**3. What does `averageUtilization: 50` mean?**

It instructs the HPA to target an average CPU utilization of 50% of the CPU requests across the Pods managed by the Deployment.

---

**4. Why should only one HPA manage a Deployment at a time?**

Only one HPA should manage a Deployment to prevent conflicting scaling decisions. Multiple HPAs managing the same Deployment could repeatedly change the replica count and cause unpredictable scaling behaviour.

---

# Task 6 — Optional: Generate CPU Load and Observe Scaling

## Goal

Generate CPU load in one NGINX Pod and observe how HPA responds.

### Evidence

If you skip this task, write “Not completed — optional” and leave the remaining Task 6 placeholders blank.

Task status: Add your status here.

#### Screenshot 11 — Terminal showing the CPU-load command running inside the selected Pod



---

#### Screenshot 12 — Output of `kubectl get hpa` and `kubectl get pods` while HPA evaluates or changes the replica count

<img width="479" height="149" alt="Ass4-ss12" src="https://github.com/user-attachments/assets/6ee7cd43-7da3-49db-ae16-2e4ec51cef83" />


---

### Notes

Complete these answers only if you perform the optional test.

**1. Did the HPA change the number of Pods? Explain what you observed.**



---

**2. If scaling did not happen, what are two possible reasons?**



---

**3. Why is automatic scale-down useful after traffic decreases?**



---

# LinkedIn Requirement

### Evidence

#### Screenshot 13 — Published LinkedIn post showing your name, the required explanation, and the attached Kubernetes screenshot



#### Linkedin Post Link


---

# Submission Instructions

- Include required Screenshots 1–10 and Screenshot 13.
- Include Screenshots 11–12 only if you complete Task 6.
- Full Name must be visible in required screenshots.
- Ensure screenshots are clear and readable.
- Include the completed `autohealing-deployment.yaml` and `hpa-deployment.yaml` files.
- Include `hpa-nginx.yaml` if you selected the declarative HPA method.
- Complete the Notes sections for Tasks 1–5 in your own words.
- Complete Task 6 notes only if you perform the optional test.
- Do not expose passwords, tokens, private keys, kubeconfig files, or account IDs.

---

# Completion Checklist

- [ ] Checked the active context and existing cluster resources
- [ ] Created the autoscaling working directory (Screenshot 1)
- [ ] Created `autohealing-deployment.yaml` (Screenshot 2)
- [ ] Confirmed two auto-healing Pods were running (Screenshot 3)
- [ ] Deleted one managed Pod (Screenshot 4)
- [ ] Confirmed that the Pod was automatically replaced (Screenshot 5)
- [ ] Created `hpa-deployment.yaml` with CPU requests and limits (Screenshot 6)
- [ ] Verified `nginx-deployment` and its running Pods (Screenshot 7)
- [ ] Verified Metrics Server and confirmed Pod metrics are available (Screenshot 8)
- [ ] Created one HPA using the selected method (Screenshot 9)
- [ ] Verified minimum replicas of 2, maximum replicas of 5, and a CPU target of 50% (Screenshot 10)
- [ ] Completed Notes for Tasks 1–5
- [ ] Completed Task 6 evidence and notes, or marked it “Not completed — optional”
- [ ] Stopped the CPU-load command and exited the container shell, if Task 6 was completed
- [ ] Published the LinkedIn post with the required explanation and attached screenshot (Screenshot 13)
- [ ] Included the required manifest files
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
