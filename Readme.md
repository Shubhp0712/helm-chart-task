# Nginx Helm Chart – Minikube Deployment

## 1. Project Structure

```text
nginx-chart/
│
├── templates/
│   ├── _helpers.tpl
│   ├── configmap.yaml
│   ├── deployment.yaml
│   └── service.yaml
│   └── serviceaccount.yaml
│
├── Chart.yaml
├── values.yaml
├── test-value.yaml
├── demo-value.yaml
├── .helmignore

README.md
```

## 2. Prerequisites

Make sure the following tools are installed:

- Minikube
- kubectl
- Helm

Start Minikube:

```bash
minikube start
```

Verify the cluster:

```bash
kubectl get nodes
```

Check Helm installation:

```bash
helm version
```
## 3. Create the Chart
```
helm create nginx-chart
```
## 4. Configuration Management

The chart uses `values.yaml` to define default configurations.

Create Environment-specific configuration files that override the default values. This is neccesary for different copies.(eg. test-value.yaml, demo-value.yaml)

| Configuration | values.yaml | test-value.yaml | demo-value.yaml |
| ------------- | ----------- | --------------- | --------------- |
| Replicas      | Default     | Test-specific   | Demo-specific   |
| Image         | Default     | Configurable    | Configurable    |
| Image version | Default     | Configurable    | Configurable    |
| Message       | Default     | Test message    | Demo message    |

This approach allows multiple application instances to use the same chart with different configurations without modifying the templates.

- example value file:
```yaml
replicaCount: 1

image:
  repository: nginx
  # This sets the pull policy for images.
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

customMessage: "<The message you want to print>"
```

## 5. Validate the Helm Chart

Run Helm's built-in lint command:

```bash
helm lint ./nginx-chart
```

Render the Test environment templates:

```bash
helm template test ./nginx-chart -f ./nginx-chart/test-value.yaml
```

Render the Demo environment templates:

```bash
helm template demo ./nginx-chart -f ./nginx-chart/demo-value.yaml
```

These commands validate the chart structure and allow inspection of the generated Kubernetes manifests.

## 6. Deploy Test and Demo Environments

### Deploy Test Environment

Install the chart using the Test configuration:

```bash
helm install test ./nginx-chart -f ./nginx-chart/test-value.yaml
```

### Deploy Demo Environment

Install the same chart using the Demo configuration:

```bash
helm install demo ./nginx-chart -f ./nginx-chart/demo-value.yaml
```

Both releases run independently in the same Kubernetes cluster.

### Verify Releases

```bash
helm list
```

Check Kubernetes resources:

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get configmaps
```

## 7. To Open web-page in browser do the port-forwarding

```bash
kubectl port-forward service/<service_name> <host_port>:<target_port>
```
Do same for both services of Test and Demo.

In browser:
```
http://localhost:<host_port>
```

## 8. Upgrade the Demo Environment

Suppose the Demo environment needs more replicas and a new webpage message.

Update the following configurations in `demo-value.yaml`:

- Increase the replica count.
- Change the custom message.

Apply the updated configuration without uninstalling the existing release:

```bash
helm upgrade demo ./nginx-chart -f ./nginx-chart/demo-value.yaml
```

Verify the updated deployment:

```bash
kubectl get deployments
kubectl get pods
```

Refresh the Demo webpage to verify the updated message.

The Test environment should remain unchanged because only the Demo release was upgraded.

## 9. Check Helm Release History

Helm maintains revision history for each release.

View the Demo release history:

```bash
helm history demo
```

Inspect the initial installation:

```bash
helm get values demo --revision 1
```

Inspect the upgraded configuration:

```bash
helm get values demo --revision 2
```

Compare the configurations to verify that the Demo release has been upgraded successfully.

Each successful upgrade creates a new release revision.

## 10. Cleanup

After completing the demonstration, uninstall both Helm releases.

Remove Test:

```bash
helm uninstall test
```

Remove Demo:

```bash
helm uninstall demo
```

Verify that the releases have been removed:

```bash
helm list
```

Check remaining application resources:

```bash
kubectl get all
kubectl get configmaps
```

If you have used additional resources, such as Ingresses or other custom resources, verify those separately as well.

The application resources created by both releases should no longer exist.