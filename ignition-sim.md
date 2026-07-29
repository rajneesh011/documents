### Step 1: Inspect the Argo CD application

Run:

```bash
kubectl get application <app-name> -n argocd -o yaml
```

Look for something like:

```yaml
spec:
  source:
    repoURL: https://github.com/company/repo.git
    path: apps/ignition/overlays/dev
    targetRevision: main
```

The important fields are:

* `repoURL` → Git repository
* `path` → Overlay directory
* `targetRevision` → Branch

### Step 2: Open the overlay

Suppose the path is:

```
apps/ignition/overlays/dev
```

You might see:

```
apps/
└── ignition/
    ├── base/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── kustomization.yaml
    └── overlays/
        ├── dev/
        │   ├── kustomization.yaml
        │   ├── patch.yaml
        │   └── secrets.yaml
        └── qa/
```

### Step 3: Create a new overlay (or copy the existing one)

If you're deploying a second instance (`ignition-sim-b`), you can copy the existing overlay:

```bash
cp -r overlays/dev overlays/dev-b
```

Then update the files.

For example, in `kustomization.yaml`:

```yaml
nameSuffix: -b
```

or update the deployment name via a patch.

### Step 4: Update the values

Based on your ticket, you'll likely need to change:

* Deployment name
* Client Secret
* License Key
* Activation Token
* Service name (if required)
* Ingress hostname (if required)

For example:

```yaml
env:
- name: LICENSE_KEY
  valueFrom:
    secretKeyRef:
      name: ignition-license-key-b

- name: ACTIVATION_TOKEN
  valueFrom:
    secretKeyRef:
      name: ignition-activation-token-b
```

### Step 5: Create a new Argo CD Application (if needed)

If each instance has its own Argo CD Application, you'll create another `Application` manifest pointing to the new overlay:

```yaml
spec:
  source:
    path: apps/ignition/overlays/dev-b
```

Then apply it:

```bash
kubectl apply -f ignition-sim-b-application.yaml
```

Argo CD will create the new deployment.

---

## I can guide you exactly if you share:

1. The output of:

```bash
kubectl get application <app-name> -n argocd -o yaml
```

(especially the `spec.source` section)

2. The repository structure, for example:

```bash
tree apps/ignition -L 3
```

or

```bash
find apps/ignition -maxdepth 3
```

3. The contents of the overlay's `kustomization.yaml`.

With those, I can tell you precisely which files to copy and which values to change to create the new `ignition-sim-b` instance.
