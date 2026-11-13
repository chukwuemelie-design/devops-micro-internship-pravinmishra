# Assignment 8 — Kubernetes Services (NodePort)

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Create a healthy NGINX Deployment, expose it through a NodePort Service, test access using a node IP and NodePort, and observe how the Service behaves when a backend Pod is replaced.

---

# Task 1 — Set Up Your Lab Directory

## Goal

Create and enter a dedicated directory for the NodePort Service lab.

### Evidence

#### Screenshot 1 — Terminal showing that you are working inside `~/k8s-labs/services/nodeport`

<img width="480" height="101" alt="Ass8-ss1" src="https://github.com/user-attachments/assets/2a4062e9-8d73-4bdf-b419-b986c627fbf9" />


### Notes

**1. What is the difference between ClusterIP and NodePort?**

ClusterIP exposes a Service internally within the Kubernetes cluster, while NodePort exposes the Service through a port on each cluster node, allowing access from outside the cluster when network connectivity permits.

**2. What does a NodePort Service allow users to do?**

It allows users to access an application from outside the cluster using a node's IP address and an assigned NodePort.

---

# Task 2 — Create a Healthy NGINX Deployment

## Goal

Create a healthy NGINX Deployment that can act as the backend for the NodePort Service.

### Evidence

#### Screenshot 2 — Completed `00-nginx-deploy.yaml` manifest

<img width="626" height="372" alt="Ass8-ss2" src="https://github.com/user-attachments/assets/0f463acc-8205-467f-a9e9-716ebe36048f" />


---

#### Screenshot 3 — Output showing successful rollout and two ready NGINX Pods

<img width="667" height="116" alt="Ass8-ss3" src="https://github.com/user-attachments/assets/7be2e6f8-98ab-4540-ad7b-bb2855bc2630" />


### Notes

**1. Which label will the NodePort Service use to find the NGINX Pods?**

The Service will use the label app: nginx to identify and select the NGINX Pods.

**2. Why must the Pods be ready before they can receive Service traffic?**

Ready Pods are considered healthy and able to handle requests. Kubernetes uses their readiness status when determining which Pods should receive Service traffic.

**3. What port does the NGINX container listen on?**

NGINX listens on port 80.

---

# Task 3 — Create the NodePort Service

## Goal

Create a NodePort Service that exposes the NGINX application through a port on every cluster node.

### Evidence

#### Screenshot 4 — Completed `01-nginx-svc-nodeport.yaml` manifest

<img width="620" height="338" alt="Ass8-ss4" src="https://github.com/user-attachments/assets/863ba61a-665e-4580-96c6-2870597b794e" />


---

#### Screenshot 5 — Output showing the NodePort Service and its endpoints or EndpointSlices

<img width="681" height="397" alt="Ass8-ss5" src="https://github.com/user-attachments/assets/ce6efdfe-2edd-4218-9230-edf48e075afc" />


### Notes

**1. What is the purpose of `type: NodePort`?**

type: NodePort exposes a Service through a port on every node in the cluster, allowing external clients to access the application using a node IP and NodePort.

**2. What is the difference between `port`, `targetPort`, and `nodePort`?**

port: The port exposed by the Kubernetes Service.
targetPort: The port on the backend container where traffic is sent.
nodePort: The port opened on each cluster node for external access to the Service.

**3. What NodePort was assigned or selected in your environment?**

Use the actual NodePort displayed by your Service. For example, if your output shows 32643, enter:

32643

**4. Why do Service endpoints need ready Pods?**

Ready Pods are capable of handling requests. Kubernetes uses readiness information to ensure that Service traffic is directed to healthy backend Pods instead of unready ones.

---

# Task 4 — Find a Node IP and Test the NodePort

## Goal

Find a reachable node IP and access the NodePort Service.

### Evidence

#### Screenshot 6 — Output of `kubectl get nodes -o wide`

<img width="920" height="186" alt="Ass8-ss6" src="https://github.com/user-attachments/assets/97172c56-af2d-4bed-8ba3-693aa0e02405" />


---

#### Screenshot 7 — Output showing the `tester` Pod running

<img width="469" height="71" alt="Ass8-ss7" src="https://github.com/user-attachments/assets/e8659b24-710e-458a-ae48-6311199b9ba2" />


---

#### Screenshot 8 — Successful NodePort response from the laptop or the `tester` Pod

If neither route is supported by your environment, include the failed request output and ready endpoint evidence, and explain the limitation in your Notes.

<img width="785" height="194" alt="Ass8-ss8" src="https://github.com/user-attachments/assets/61ccda64-1e6a-4e30-881d-de32441c3850" />


### Notes

**1. Which node IP and NodePort did you test?**

Use the actual node IP and NodePort from your terminal output. For example, if your node IP was 192.168.49.2 and your NodePort was 32643, enter:

192.168.49.2:32643

**2. Was your laptop able to reach the NodePort directly? Explain your result.**

If successful: Yes. My laptop successfully accessed the NGINX application using the node IP and NodePort.

**3. Why can external NodePort access behave differently in Minikube, kind, Docker Desktop, and cloud clusters?**

These Kubernetes environments use different networking architectures. Minikube may require a tunnel or special access method, kind runs nodes inside Docker containers, Docker Desktop uses its own networking layer, and cloud clusters may expose node IPs through public or private networks controlled by firewall and security-group rules.

---

# Task 5 — Optional: Test the Label-Selector Contract

## Goal

Break the Service selector, observe that usable endpoints disappear, and restore the correct selector.

Complete the Evidence and Notes below only if you attempted this optional task.

### Evidence

#### Screenshot 9 — `02-nginx-svc-selector-broken.yaml` showing the incorrect selector

<img width="814" height="331" alt="Ass8-ss9" src="https://github.com/user-attachments/assets/2e8ad2f1-b590-4471-8e27-9a3945f6ba7d" />


---

#### Screenshot 10 — Empty endpoints or EndpointSlice output after applying the broken selector

<img width="869" height="278" alt="Ass8-ss10" src="https://github.com/user-attachments/assets/aaf4b117-f73f-4338-ad9e-2c3a1e351345" />


---

#### Screenshot 11 — Restored endpoints or EndpointSlices after applying the original Service manifest

<img width="848" height="343" alt="Ass8-ss11" src="https://github.com/user-attachments/assets/603e82bd-e42c-4fe6-bd5e-5ecf3d814f13" />


### Notes

**1. Why did the Service endpoints become empty?**

The Service selector was changed to an incorrect label that did not match the NGINX Pod labels. Kubernetes could no longer identify matching backend Pods, so the Service had no usable endpoints.

**2. Why is NodePort only useful when matching ready Pods exist?**

NodePort provides an external entry point, but it still needs healthy backend Pods to process incoming requests. Without matching ready Pods, there are no usable backends to receive the traffic.

**3. How did you restore the backend connectivity?**

I restored the original Service selector to match the NGINX Pod label, app: nginx, and reapplied the correct Service manifest. Kubernetes then restored the Service endpoints and backend connectivity.

---

# Task 6 — Observe Resilience During Pod Churn

## Goal

Delete one NGINX Pod and observe its replacement while the Service continues routing traffic to available Pods.

### Evidence

#### Screenshot 12 — Output of the Pod deletion command

<img width="632" height="74" alt="Ass8-ss12" src="https://github.com/user-attachments/assets/ff4236cf-9bb2-42d4-b371-a71a139129a8" />


---

#### Screenshot 13 — Output showing the replacement Pod and two running, ready Pods

<img width="526" height="98" alt="Ass8-13" src="https://github.com/user-attachments/assets/5a2809a0-56d7-4aae-9fa2-78395adbf147" />


### Notes

**1. What happened after you deleted one Pod?**

Kubernetes detected that the Deployment had fewer Pods than its desired replica count and automatically created a replacement Pod. The Deployment returned to two running, ready Pods.

**2. Which Kubernetes resource created the replacement Pod?**

The ReplicaSet controller created the replacement Pod on behalf of the Deployment.

**3. Why can the NodePort Service continue working during Pod churn?**

The NodePort Service provides a stable access point, while Kubernetes automatically updates its backend endpoints as Pods are replaced. Traffic can continue reaching the remaining ready Pods during the replacement process.

---

# Task 7 — Share Your NodePort Learning on LinkedIn

## Goal

Share what you learned about NodePort Services, backend Pods, and network access in your Kubernetes environment.

### Evidence

#### Screenshot 14 — Published LinkedIn post about your NodePort Service lab



#### Linkedin post link



---

# Submission Instructions

- Include the completed YAML manifests created during the lab.
- Add all required screenshots: Screenshots 1–8 and 12–14.
- Include Screenshots 9–11 only if you attempted optional Task 5.
- Complete the Notes for Tasks 1–4 and Task 6.
- Complete Task 5 Notes only if you attempted that task.
- Use clear, readable screenshots showing the relevant commands and outputs.
- Ensure your full name is visible in the required screenshots.
- If NodePort access is restricted by your environment, include the failed request output, ready endpoint evidence, and an explanation.
- Do not expose passwords, access tokens, private keys, kubeconfig contents, or account IDs.

---

# Completion Checklist

- [ ] Created and entered `~/k8s-labs/services/nodeport` (Screenshot 1)
- [ ] Created and applied the NGINX Deployment manifest (Screenshot 2)
- [ ] Confirmed successful rollout and two ready NGINX Pods (Screenshot 3)
- [ ] Created the NodePort Service manifest (Screenshot 4)
- [ ] Verified the Service and its endpoints or EndpointSlices (Screenshot 5)
- [ ] Recorded the assigned or selected NodePort
- [ ] Identified a node IP (Screenshot 6)
- [ ] Confirmed that the `tester` Pod is running (Screenshot 7)
- [ ] Tested NodePort access, or documented the environment limitation with evidence (Screenshot 8)
- [ ] Optional: Tested and restored the Service selector (Screenshots 9–11)
- [ ] Deleted one NGINX Pod (Screenshot 12)
- [ ] Confirmed the replacement Pod and two running, ready Pods (Screenshot 13)
- [ ] Completed all required Notes
- [ ] Published the LinkedIn post (Screenshot 14)
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
