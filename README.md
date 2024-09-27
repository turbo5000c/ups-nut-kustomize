# UPS NUT Kustomize Manifests

This repository contains a Kustomize setup for deploying a UPS NUT (Network UPS Tools) StatefulSet. The Kustomize manifests provide a simple and customizable way to deploy the StatefulSet along with configuration options for a ConfigMap and a Secret.

## Prerequisites

- Kubernetes cluster (v1.18+ recommended)
- Kustomize installed on your local machine (`kubectl kustomize` can be used)
- Helm or kubectl for deploying the chart

## Install the Kustomize Manifests

To install the manifests using Kustomize, follow these steps:

1. Clone the repository:
    ```bash
    git clone https://github.com/turbo5000c/ups-nut-kustomize.git
    cd ups-nut-kustomize
    ```

2. Deploy the manifests:
    ```bash
    kubectl apply -k .
    ```

This command will apply all Kustomize configurations and deploy the StatefulSet along with the required ConfigMap and Secret.

## Customizing the Configuration

### ConfigMap

The ConfigMap is used to configure UPS NUT settings such as drivers, ups.conf, and other necessary configurations.

1. Open the `configmap.yaml` file located in the `kustomize` directory:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: ups-nut-config
    data:
      NUT_DRIVER: "usbhid-ups" 
    ```

2. Modify the `ups.conf` or other configuration data as needed to match your UPS configuration.

3. Apply the changes:
    ```bash
    kubectl apply -f configmap.yaml
    ```

### Secret

The Secret is used to store sensitive information like credentials for NUT access.

1. Open the `secret.yaml` file located in the `kustomize` directory:
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: ups-nut-secret
    type: Opaque
    data:
      API_USER: changeme
      API_PASSWORD: changeme
    ```

2. Encode your credentials in base64 format:
    ```bash
    echo -n 'API_USER: changeme' | base64
    echo -n 'API_PASSWORD: changeme' | base64
    ```

3. Replace the `<base64-encoded-username>` and `<base64-encoded-password>` with your actual encoded values.

4. Apply the changes:
    ```bash
    kubectl apply -f secret.yaml
    ```

## Uninstall

To uninstall the StatefulSet and related resources:
```bash
kubectl delete -k .
```
