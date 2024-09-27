# UPS NUT Kustomize Manifests

This repository contains a Kustomize setup for deploying a UPS NUT (Network UPS Tools) StatefulSet. The Kustomize manifests provide a simple and customizable way to deploy the StatefulSet along with configuration options for a ConfigMap, a Secret, and USB device mapping.

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

This command will apply all Kustomize configurations and deploy the StatefulSet along with the required ConfigMap, Secret, and USB device mapping.

## Customizing the Configuration

### ConfigMap

The ConfigMap is used to configure UPS NUT settings such as the driver type.

1. Open the `configmap.yaml` file located in the `kustomize` directory:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nut-config
      namespace: default
    data:
      NUT_DRIVER: "usbhid-ups"
    ```

2. Modify the `NUT_DRIVER` value if needed to match your UPS setup.

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
      name: nut-secrets
    type: Opaque
    stringData:
      API_USER: changeme
      API_PASSWORD: changeme
    ```

2. Update the `API_USER` and `API_PASSWORD` with your actual credentials.

3. Apply the changes:
    ```bash
    kubectl apply -f secret.yaml
    ```

### USB Device Mapping

The StatefulSet requires access to the UPS device via USB, which is configured using a `hostPath` volume.

1. Open the `statefulset.yaml` file and locate the section for USB device mapping:
    ```yaml
    volumeMounts:
    - name: usb-device
      mountPath: /dev/bus/usb
    volumes:
    - name: usb-device
      hostPath:
        path: /dev/bus/usb/002/009 # Update this for your environment
        type: CharDevice
    ```

2. Update the `hostPath` to point to the correct USB path for your environment:
    - Use `lsusb` on your host to determine the correct bus and device number for your UPS.
    - Example:
      ```bash
      lsusb
      Bus 002 Device 009: ID 1234:5678 Your UPS Manufacturer
      ```

    - In this example, you would update the path to `/dev/bus/usb/002/009`.

3. Apply the updated manifests:
    ```bash
    kubectl apply -f statefulset.yaml
    ```

## Using Kustomize Patches

To avoid manually modifying the base manifests for different environments, you can create Kustomize patches to handle customization.

### Create a Patch for ConfigMap

1. Create a file called `configmap-patch.yaml`:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nut-config
    data:
      NUT_DRIVER: "usbhid-ups"
    ```

2. Update the `NUT_DRIVER` if necessary.

### Create a Patch for Secret

1. Create a file called `secret-patch.yaml`:
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: nut-secrets
    stringData:
      API_USER: newuser
      API_PASSWORD: newpassword
    ```

2. Update the `API_USER` and `API_PASSWORD` values as needed.

### Create a Patch for USB Device Mapping

1. Create a file called `usb-patch.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: ups-nut
    spec:
      template:
        spec:
          volumes:
          - name: usb-device
            hostPath:
              path: /dev/bus/usb/<your-device-path>
              type: CharDevice
    ```

2. Replace `<your-device-path>` with the correct path for your USB device.

### Add Patches to Kustomize

1. Open the `kustomization.yaml` file and add the patches:
    ```yaml
    patchesStrategicMerge:
      - configmap-patch.yaml
      - secret-patch.yaml
      - usb-patch.yaml
    ```

2. Apply the manifests with the patches:
    ```bash
    kubectl apply -k .
    ```

## Uninstall

To uninstall the StatefulSet and related resources:
```bash
kubectl delete -k .
```
