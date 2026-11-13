# Assignment 5 — Kubernetes Health Probes (Readiness)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will configure and test a Kubernetes readiness probe. You will create a baseline NGINX Deployment, add an HTTP readiness probe, intentionally break it, observe a stalled rollout, and restore the working configuration.

You will also demonstrate how readiness probes protect rolling updates.

---

# Task 0 — Prepare the Lab Environment

## Goal

Confirm that you are connected to your learning cluster and remove the previous assignment’s HPA and NGINX Deployment before starting this lab.

No screenshot or written notes are required for this pre-check.

---

# Task 1 — Set Up Your Lab Directory

## Goal

Create and enter a dedicated directory for the readiness-probe lab.

### Evidence

#### Screenshot 1 — Terminal showing that you are working inside `~/k8s-labs/health-probes-readiness`

<img width="410" height="137" alt="Ass5-ss1" src="https://github.com/user-attachments/assets/e0af4f91-da70-4797-b994-cf8cd41aa59f" />


---

### Notes

**1. What is the purpose of a readiness probe?**

A readiness probe checks whether a container is ready to accept traffic. Kubernetes uses the result to determine whether the Pod should receive traffic through its Service.

**2. Why is a Pod being Running not always enough to confirm it can serve traffic?**

Running only means the container has started. The application inside it may still be initializing, loading dependencies, or experiencing errors. A readiness probe confirms whether it is actually ready to handle requests.

---

# Task 2 — Create a Baseline Deployment Without Probes

## Goal

Create a baseline NGINX Deployment with two replicas and no readiness probe.

### Evidence

#### Screenshot 2 — Completed `00-nginx-deploy-baseline.yaml` manifest

<img width="563" height="306" alt="Ass5-ss2" src="https://github.com/user-attachments/assets/ae2a4ef1-62b5-4a40-815d-1944f69a4f5e" />


---

#### Screenshot 3 — Output showing a successful rollout and two running NGINX Pods

<img width="651" height="124" alt="Ass5-ss3" src="https://github.com/user-attachments/assets/be52cbbd-78ef-4d03-b433-b6f839b7e1bf" />


---

#### Screenshot 4 — Relevant output from `kubectl describe deployment nginx-deployment`

<img width="872" height="217" alt="Ass5-ss4" src="https://github.com/user-attachments/assets/c83a4490-5662-4f62-a86a-85effa4f86c1" />


---

### Notes

**1. How many replicas does this Deployment maintain?**

The Deployment maintains two replicas of the NGINX Pod.

**2. What does `maxUnavailable: 0` mean during a rolling update?**

It means Kubernetes must not allow the number of available Pods to fall below the desired replica count during a rolling update. Existing healthy Pods remain available while replacements are created.

**3. Is a readiness probe configured in this baseline Deployment?**

No. The baseline Deployment does not have a readiness probe configured.

---

# Task 3 — Add a Readiness Probe

## Goal

Add an HTTP readiness probe so Kubernetes checks whether each NGINX container is ready to accept requests.

### Evidence

#### Screenshot 5 — Completed `01-nginx-deploy-readiness.yaml` manifest showing the readiness probe

<img width="554" height="374" alt="Ass5-ss5" src="https://github.com/user-attachments/assets/85f74e84-661b-4970-8b3c-84a692b928cd" />


---

#### Screenshot 6 — Output of the successful rollout and ready Pods

<img width="676" height="80" alt="Ass5-ss6" src="https://github.com/user-attachments/assets/a0eaa154-1f7c-42a4-bdd0-887eb737f719" />


---

#### Screenshot 7 — Pod description showing readiness conditions and readiness probe details

<img width="613" height="118" alt="Ass5-ss7" src="https://github.com/user-attachments/assets/be309b5a-3167-41af-963e-acea9a4555b9" />


---

### Notes

**1. What path and port does the readiness probe check?**

It checks the HTTP path / on port 80.

**2. What does `initialDelaySeconds: 2` do?**

It tells Kubernetes to wait two seconds after the container starts before performing the first readiness probe.

**3. What does `periodSeconds: 5` do?**

It tells Kubernetes to perform the readiness check every five seconds.

**4. What must happen before Kubernetes considers this Pod ready?**

The NGINX container must respond successfully to the HTTP readiness check on the configured path and port. Kubernetes then marks the Pod as Ready.

---

# Task 4 — Break and Restore Readiness

## Goal

Intentionally fail the readiness probe, observe the NotReady behavior, and then restore the working probe configuration.

### Evidence

#### Screenshot 8 — `02-nginx-readiness-broken.yaml` showing `/does-not-exist`

<img width="696" height="112" alt="Ass5-ss8" src="https://github.com/user-attachments/assets/68fe0657-27a3-4ae1-9d03-fa158ca64121" />


---

#### Screenshot 9 — Output showing the rollout timeout or incomplete rollout

<img width="725" height="77" alt="Ass5-ss9" src="https://github.com/user-attachments/assets/9f19b331-c5d3-4654-8873-b2f828b23acb" />


---

#### Screenshot 10 — Pod details showing `Ready: False` or readiness-probe failure events

<img width="764" height="394" alt="Ass5-ss10" src="https://github.com/user-attachments/assets/2dbd9bbc-7831-4f7d-b9a6-f032f7aaca42" />


---

#### Screenshot 11 — Output showing successful rollout and ready Pods after the fix

<img width="658" height="91" alt="Ass5-ss11" src="https://github.com/user-attachments/assets/5170abae-8628-4ea4-8cb9-42518767c06b" />


---

### Notes

**1. Why did the readiness probe fail?**

The readiness probe failed because it was configured to check /does-not-exist, a path that NGINX does not serve successfully. The HTTP request returned an unsuccessful response.

**2. What happened to the Pod readiness status?**

The Pod remained Running, but its readiness status changed to False. Kubernetes therefore treated it as not ready to receive traffic.

**3. Why did the rollout fail to complete?**

The rollout could not complete because the new Pod failed its readiness checks. Kubernetes waited for a ready replacement before proceeding with the rolling update.

**4. How did you restore the Deployment successfully?**

I restored the correct readiness probe path to / on port 80 and applied the corrected Deployment configuration. The readiness checks then succeeded, and the rollout completed successfully.

---

# Task 5 — See How Readiness Protects Rolling Updates

## Goal

Observe how readiness prevents a bad new Pod from replacing healthy Pods during a rolling update.

### Evidence

#### Screenshot 12 — Successful update command and rollout status

<img width="747" height="141" alt="Ass5-ss12" src="https://github.com/user-attachments/assets/0db19b44-d434-41a3-9e61-2a06519558c9" />


---

#### Screenshot 13 — ReplicaSets and Pods after the successful rolling update

<img width="906" height="202" alt="Ass5-ss13" src="https://github.com/user-attachments/assets/f4a93b55-66c2-4af9-9658-460203324ad8" />


---

#### Screenshot 14 — Completed `04-readiness-broken-patch.yaml`

<img width="580" height="248" alt="Ass5-ss14" src="https://github.com/user-attachments/assets/b10b8440-0b5c-4ab4-a315-987cfbd26554" />


---

#### Screenshot 15 — Output showing the expected stalled rollout or timeout

<img width="737" height="89" alt="Ass5-ss15" src="https://github.com/user-attachments/assets/1ac26098-4f2d-46b9-82a2-56b143bd5453" />


---

#### Screenshot 16 — Deployment or Pod output showing the effect of the broken readiness probe

<img width="641" height="188" alt="Ass5-ss16" src="https://github.com/user-attachments/assets/ae172639-908b-405e-bef3-7daa2e02caab" />


---

#### Screenshot 17 — Successful rollout status after restoring the fixed configuration

<img width="692" height="227" alt="Ass5-ss17" src="https://github.com/user-attachments/assets/c792444b-b601-4bac-b7ca-8527cd2b1d84" />


---

### Notes

**1. What happened during the successful rolling update?**

Kubernetes gradually created new Pods, waited for them to become Ready, and then replaced the old Pods. The application remained available throughout the update.

**2. Why did Kubernetes create a new ReplicaSet?**

A change to the Deployment's Pod template triggered Kubernetes to create a new ReplicaSet. The new ReplicaSet manages Pods using the updated configuration.

**3. Why were the new Pods unable to replace the existing healthy Pods after readiness was broken?**

The new Pods failed their readiness probes and were not considered available. Kubernetes therefore kept the existing healthy Pods running instead of replacing them with unready Pods.

**4. How does `maxUnavailable: 0` protect availability during this rollout?**

It prevents Kubernetes from reducing the number of available Pods below the desired replica count during the rolling update. This helps ensure that healthy Pods remain available while new Pods are being tested.

---

# Task 6 — Share Your Readiness-Probe Learning on LinkedIn

## Goal

Share what you learned about Kubernetes readiness probes and how they protect rolling updates.

### Evidence

#### Screenshot 18 — Published LinkedIn post



#### Linkedin post link


---

# Submission Instructions

- Add all required screenshots numbered 1–18.
- Ensure your full name is visible in required screenshots.
- Use clear, readable screenshots showing the relevant manifest or output.
- Include the completed baseline and readiness-probe manifests in your repository.
- Answer all Notes questions for Tasks 1–5 in your own words.
- Include evidence of the intentional readiness failure and successful recovery.
- Do not expose tokens, passwords, kubeconfig files, account IDs, or other sensitive information.

---

# Completion Checklist

- [ ] Learning cluster confirmed and previous assignment resources removed where applicable
- [ ] Readiness-probe lab directory created (Screenshot 1)
- [ ] Baseline Deployment created successfully (Screenshot 2)
- [ ] Two baseline Pods running and successful rollout verified (Screenshot 3)
- [ ] Baseline Deployment details reviewed (Screenshot 4)
- [ ] Readiness probe added successfully (Screenshot 5)
- [ ] Successful readiness rollout verified (Screenshot 6)
- [ ] Pod readiness conditions and probe details inspected (Screenshot 7)
- [ ] Broken readiness configuration created (Screenshot 8)
- [ ] Stalled rollout or timeout observed (Screenshot 9)
- [ ] NotReady status or readiness-probe failure events captured (Screenshot 10)
- [ ] Working readiness configuration restored (Screenshot 11)
- [ ] Successful image update completed (Screenshot 12)
- [ ] ReplicaSets and Pods inspected after the update (Screenshot 13)
- [ ] Broken-readiness patch created (Screenshot 14)
- [ ] Stalled rollout observed after the patch (Screenshot 15)
- [ ] Effect of broken readiness captured (Screenshot 16)
- [ ] Final recovery completed (Screenshot 17)
- [ ] LinkedIn post published (Screenshot 18)
- [ ] Notes for Tasks 1–5 completed
- [ ] Required manifests included
- [ ] All screenshots are clear and readable
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
