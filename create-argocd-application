# Hands-On: Creating Your First Argo CD Application

Deploy a static NGINX application into a Kubernetes cluster using **Argo CD** and **GitOps** principles.

---

## 1. Prepare the Kubernetes Manifests

### Step 1: Create a Working Directory
```bash
mkdir argocd-demo-app
cd argocd-demo-app
```

### Step 2: Create `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Step 3: Create `service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-demo
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### Step 4: Push Manifests to GitHub
Create a new **public** repository on GitHub named `argocd-demo`, then push your files:

```bash
git init
git branch -M main
git add .
git commit -m "Add nginx deployment and service"
git remote add origin https://github.com/YOUR-USERNAME/argocd-demo.git
git push -u origin main
```
> **Note:** Replace `YOUR-USERNAME` with your actual GitHub username.

---

## 2. Deploy the Application using Argo CD

Choose **one** of the three methods below to create and deploy the application.

### Method 1: Using the Argo CD Web UI

1. **Connect Repository:**
   - Log in to the Argo CD Web UI at `https://localhost:8080`.
   - Navigate to **Settings** $\rightarrow$ **Repositories** $\rightarrow$ **Connect Repository**.
   - Select **via HTTPS**, enter your Git Repo URL (`https://github.com/YOUR-USERNAME/argocd-demo.git`), and click **Connect**.

2. **Create Application:**
   - Go to **Applications** $\rightarrow$ **New App**.
   - **Application Name:** `argocd-demo`
   - **Project Name:** `default`
   - **Sync Policy:** `Manual`
   - **Repository URL:** `https://github.com/YOUR-USERNAME/argocd-demo.git`
   - **Revision:** `HEAD`
   - **Path:** `.`
   - **Cluster URL:** `https://kubernetes.default.svc`
   - **Namespace:** `default`
   - Click **Create**.

3. **Sync Application:**
   - Click the `argocd-demo` application card.
   - Click **Sync** $\rightarrow$ **Synchronize**.
   - Wait for the status to transition from `Progressing` to `Healthy` and `Synced`.

---

### Method 2: Using the Argo CD CLI

1. **Create the Application:**
   ```bash
   argocd app create argocd-demo \
     --repo https://github.com/YOUR-USERNAME/argocd-demo.git \
     --path . \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace default
   ```

2. **Sync the Application:**
   ```bash
   argocd app sync argocd-demo
   ```

3. **Inspect Application Status:**
   ```bash
   # List all applications
   argocd app list

   # Get detailed application information
   argocd app get argocd-demo
   ```

4. **Delete Application (Optional):**
   ```bash
   argocd app delete argocd-demo
   ```

---

### Method 3: Using a Declarative YAML Manifest

1. **Create `argocd-app.yaml`:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: argocd-demo
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/YOUR-USERNAME/argocd-demo.git
       targetRevision: HEAD
       path: .
     destination:
       server: https://kubernetes.default.svc
       namespace: default
   ```

2. **Apply the Manifest:**
   ```bash
   kubectl apply -f argocd-app.yaml
   ```

3. **Sync via CLI or Web UI:**
   ```bash
   argocd app sync argocd-demo
   ```

---

## 3. Verify Kubernetes Deployment

Check that the resources were deployed to your cluster in the `default` namespace:

```bash
kubectl get all
```

---

## 4. GitOps in Action: Scale the Application

Demonstrate GitOps automation by letting Git commit changes drive your cluster state.

### Step 1: Enable Auto-Sync
* **Via Web UI:** Open `argocd-demo` $\rightarrow$ **App Details** $\rightarrow$ Click **Enable Auto-Sync**.
* **Via CLI:**
  ```bash
  argocd app set argocd-demo --sync-policy automated
  ```

### Step 2: Update Replica Count in Git
Edit `deployment.yaml` and change `replicas` from `2` to `3`:

```yaml
spec:
  replicas: 3
```

Commit and push the change:
```bash
git add deployment.yaml
git commit -m "Scale nginx to 3 replicas"
git push origin main
```

### Step 3: Verify Automated Rollout
Argo CD automatically detects the Git push and scales the deployment:

```bash
kubectl get pods -l app=nginx-demo
```

---

## 5. Troubleshooting Common Issues

### Application Stuck in `Progressing` State
Inspect deployment configuration, typos, or container image errors:
```bash
# Check application events
argocd app get argocd-demo

# Check Kubernetes deployment events
kubectl describe deployment nginx-demo
```

### Sync Failures
Investigate permissions, network connectivity, or manifest syntax errors:
```bash
# View sync operation details
argocd app get argocd-demo

# View Argo CD controller logs
kubectl logs -n argocd deployment/argocd-application-controller
```

### `Out of Sync` Status
Compare desired Git state against actual live cluster state:
```bash
# Check diff between Git and Cluster
argocd app diff argocd-demo

# Force a hard refresh
argocd app get argocd-demo --refresh
```
