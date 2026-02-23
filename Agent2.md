summary: Secure Sandbox Execution for Antigravity on GKE
id: secure-agent-sandbox-gke
categories: AI, GKE, Security
status: Published
authors: Geeta Kakrani
Feedback Link: https://www.google.com/search?q=https://github.com/geetakakrani


# **Running Agent Code Safely: Secure Sandbox Execution for Antigravity on GKE**

## **Introduction: The Need for a "Safe Room"**

Duration: 5:00

When we build agents using **Antigravity**, we are giving an AI the power to think and act. Often, that "act" involves writing and running Python code to process complex data.

### **The Problem**

If the AI generates a script that contains a bug—or if it is tricked into running a malicious command—it could potentially delete files in your Cloud Project or steal your database credentials.

### **The Solution: The GKE Sandbox**

We use **Google Kubernetes Engine (GKE)** to build a digital "Safe Room" (Sandbox). By using a technology called **gVisor**, we ensure that the code is physically isolated. Even if the code tries to "break out" and attack the computer it is running on, the Sandbox intercepts the attack and shuts it down.

## **Defining the Pillars**

Duration: 5:00

To understand this Codelab, you need to know three core components:

### **Antigravity (The Brain)**

This is your AI Agent framework. It handles the logic, talks to the LLM (like Gemini), and decides when it's time to run a script.

### **GKE (The Foundation)**

Google Kubernetes Engine is a platform that manages "Containers." Think of a container as a small, portable box that holds everything your code needs to run. GKE is the manager that decides which server those boxes should sit on.

### **The Sandbox (The Guard)**

Normally, all containers on a server share the same "operating system kernel" (the brain of the computer). A Sandbox (specifically **gVisor**) creates a "fake" brain for each container. This way, the container cannot see or touch the real computer's brain.

## **Step 1: Preparing your GCP Environment**

Duration: 5:00

We will use **Google Cloud Shell** for this lab. It is a pre-configured terminal in the cloud that already has the tools we need.

### **Action: Initialise your project**

1. Open the [Google Cloud Console](https://console.cloud.google.com/).  
2. Click the **Activate Cloud Shell** icon (`>_`) in the top right corner.  
3. Run the following command to ensure your project is set:

gcloud config set project \[YOUR\_PROJECT\_ID\]

## **Step 2: Creating the Sandbox-Enabled Cluster**

Duration: 10:00

We need to build a GKE cluster and tell it to prepare the servers with "Security Glass" (gVisor).

### **Action: Create the Cluster**

Run this command in your Cloud Shell:

gcloud container clusters create "antigravity-secure-cluster" \\

   gcloud container clusters create antigravity-secure-cluster \
  --region us-central1 \
  --num-nodes=1 \
  --machine-type=e2-medium \
  --enable-sandbox \
  --workload-pool=$(gcloud config get-value project).svc.id.goog

### **What is happening here?**

* **\--region "us-central1"**: We are choosing where the servers physically live.  
* **\--sandbox type=gvisor**: This is the most important part. It installs the protective layer that isolates our AI agents.

## **Step 3: Setting the "Safe Room" Rules**

Duration: 5:00

In Kubernetes, we use a **RuntimeClass**. Think of this as a "House Rule" that tells the system: "Whenever I label a task as 'Secure,' use the Sandbox guard."

### **Action: Create the Rule**

Create a file named `runtime.yaml`:

apiVersion: node.k8s.io/v1

kind: RuntimeClass

metadata:

  name: antigravity-sandbox

handler: gvisor

Apply the rule to your cluster:

kubectl apply \-f runtime.yaml

## **Step 4: Building the Executor Room (The Pod)**

Duration: 5:00

Now we build the actual "Room" (called a **Pod**) where your Antigravity Agent will send its code.

### **Action: Create the Pod**

Create a file named `executor.yaml`:

apiVersion: v1

kind: Pod

metadata:

  name: secure-executor-pod

  labels:

    app: agent-worker

spec:

  runtimeClassName: antigravity-sandbox \# This triggers the gVisor isolation

  containers:

  \- name: python-interpreter

    image: python:3.11-slim

    command: \["sleep", "infinity"\] \# This keeps the pod alive and waiting

    resources:

      limits:

        memory: "256Mi" \# Limits how much memory the AI can use

        cpu: "500m"

Apply the Pod:

kubectl apply \-f executor.yaml

## **Step 5: Testing the Security**

Duration: 5:00

Now, let's pretend to be the Antigravity Agent. We will send a Python script into the room and ask it to describe the computer it sees.

### **Action: Run the Check**

kubectl exec secure-executor-pod \-- python3 \-c "import os; print(os.uname())"

### **The Discovery**

Look closely at the output. You will see the word **"gVisor"**. This proves that the Python script is **NOT** seeing your actual Google Cloud server; it is seeing the "fake brain" created by the Sandbox. The isolation is successful\!

## **Step 6: Locking the Door (Network Lockdown)**

Duration: 5:00

Even if the room is secure, the AI might try to "call home" and send your data to an unknown website. We will apply a **Network Policy** to block all internet access for this room.

### **Action: Create the Lock**

Create a file named `network-lock.yaml`:

apiVersion: networking.k8s.io/v1

kind: NetworkPolicy

metadata:

  name: deny-all-egress

spec:

  podSelector:

    matchLabels:

      app: agent-worker

  policyTypes:

  \- Egress

  \# Empty Egress means nothing can leave the pod

Apply the lock:

kubectl apply \-f network-lock.yaml

## **Conclusion & Summary**

Duration: 2:00

Congratulations\! You have built a **"Zero-Trust"** environment for your AI Agent.

* **Antigravity** writes the code.  
* **GKE** provides the ground to run it.  
* **gVisor** ensures that the code cannot attack the system.  
* **Network Policies** ensure the code cannot leak your data.

Keep exploring secure execution to make your AI agents enterprise-ready\!

