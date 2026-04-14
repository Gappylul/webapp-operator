# webapp-operator

A custom Kubernetes Operator designed for Raspberry Pi clusters (k3s) to automate the deployment of web applications using the `WebApp` Custom Resource.

## Description
This operator manages the lifecycle of web applications on k3s. It watches for `WebApp` resources and automatically generates:
* **Deployment**: Runs your container image with specified replicas and environment variables.
* **Service**: A ClusterIP service mapping port `80` to container port `8080`.
* **Ingress**: Configured specifically for **Traefik** (k3s default), handling routing based on your provided `host`.

The controller is optimized for lightweight environments like the Raspberry Pi, ensuring your web stack is consistent and self-healing.

---

## Technical Specifications (CRD)

The `WebApp` resource (`platform.gappy.hu/v1`) supports the following configuration:

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `image` | `string` | **Yes** | The container image (Ensure it is compatible with ARM/Pi architecture). |
| `host` | `string` | **Yes** | The DNS host (e.g., `myapp.local` or a Pi-hole entry). |
| `replicas` | `int32` | No | Number of desired pods (Default: `1`). |
| `env` | `[]EnvVar` | No | List of environment variables for the application. |

---

## Local Development & Deployment (k3s)

Since this project is running on **k3s (Raspberry Pi)**, use the following workflow to generate manifests and deploy.

### 1. Generate Manifests
Generate the Custom Resource Definitions (CRDs) and RBAC configurations:

```sh
make manifests
```

### 2. Build the Operator
Build the controller binary:

```sh
make build
```

### 3. Deploy to k3s
After building, follow these steps to apply the configuration to your Pi cluster:
**Install the CRDs:**

```sh
kubectl apply -f config/crd/bases/platform.gappy.hu_webapps.yaml
```

**Run the Operator:**
You can run the operator directly on your Pi (or as a deployment) to start reconciling resources:

```sh
# To run locally for testing:
./bin/manager
```

### 4. Create a WebApp Instance
Create a file named `my-app.yaml`:
```yaml
apiVersion: platform.gappy.hu/v1
kind: WebApp
metadata:
  name: pi-web-app
spec:
  image: nginx:alpine # Or your custom ARM-based image
  replicas: 2
  host: pi-app.local
  env:
    - name: APP_COLOR
      value: "raspberry-red"
```

Apply it using kubectl:

```sh
kubectl apply -f my-app.yaml
```

### k3s Specific Notes

- **Traefik Integration**: This operator automatically adds the annotation\
`traefik.ingress.kubernetes.io/router.entrypoints: web` to the generated Ingress,\
matching the default k3s Traefik configuration.
- **Architecture**: Ensure the `image` defined in your `WebApp` spec is compatible with\
`arm64` or `armv7` depending on your Raspberry Pi model.

### Troubleshooting
Check the operator logs to see the reconciliation process:

```sh
kubectl logs -l control-plane=controller-manager -n webapp-operator-system
```

### License
Copyright 2026. Licensed under the Apache License, Version 2.0.