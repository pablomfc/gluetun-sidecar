## Automatically Inject Gluetun with Kyverno ClusterPolicy

This guide demonstrates how to automatically inject Gluetun as a sidecar container into deployments using a Kyverno cluster policy. Kyverno acts as an admission control, enforcing policies on kubernetes resources before they are admitted into the cluster.

**Prerequisites**

* A Kubernetes cluster (Amazon EKS was used in this example)

**Steps**

1. **Install Kyverno**

   ```bash
   helm repo add kyverno https://kyverno.github.io/kyverno/
   helm repo update
   helm install kyverno --namespace kyverno kyverno/kyverno --create-namespace
   ```

2. **Create Gluetun Configuration Secret**

   Create a Kubernetes secret named `vpn` in the `default` namespace to store Gluetun configuration details. Replace the placeholders with your actual values:

   ```bash
   kubectl create secret generic vpn -n default \
       --from-literal=FIREWALL_OUTBOUND_SUBNETS='10.128.0.0/16,172.20.0.0/16' \
       --from-literal=VPN_SERVICE_PROVIDER='your_vpn_service_provider' \
       --from-literal=OPENVPN_USER='your_openvpn_username' \
       --from-literal=OPENVPN_PASSWORD='your_openvpn_password'
   ```

   * The `FIREWALL_OUTBOUND_SUBNETS` option specifies comma-separated network CIDRs that should not be routed through the VPN.

3. **Create Kyverno ClusterPolicy**

   Apply the `kyverno-clusterpolicy.yaml` file containing the Kyverno policy definition. This policy will look for deployments with the annotation `vpn/gluetun-inject: "true"` and automatically inject the Gluetun sidecar container.

   ```bash
   kubectl apply -f kyverno-clusterpolicy.yaml
   ```

4. **Test with Deployment Example**

   Deploy a test deployment with the `vpn/gluetun-inject: "true"` annotation in the `deployment.yaml` file and apply it to the default namespace. Kyverno will automatically inject the Gluetun container during deployment.

   ```bash
   kubectl apply -f deployment.yaml -n default
   ```

This process automates Gluetun injection for deployments marked for the sidecar container, simplifying your configuration management and ensuring consistent VPN protection for your workloads.
