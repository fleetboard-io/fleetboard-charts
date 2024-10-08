## Usage

[Helm](https://helm.sh) must be installed to use the charts. Please refer to Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

```bash
helm repo add fleetboard https://fleetboard-io.github.io/fleetboard-charts
```

If you had already added this repo earlier, run `helm repo update` to retrieve the latest versions of the packages. You can then run `helm search repo fleetboard` to see the charts.

### Install/Uninstall FleetBoard on Hub Cluster

The **Hub Cluster** is responsible for managing child clusters and establishing cross-cluster communication. The hub handles service synchronization and VPN tunnel management.

To install the `fleetboard` chart on the hub:

```bash
helm install fleetboard fleetboard/fleetboard --namespace fleetboard-system --create-namespace \
--set tunnel.endpoint=<Hub Public IP> --set tunnel.cidr=20.112.0.0/12
```

- **tunnel.endpoint**: This is the public IP of your hub cluster. The child clusters will connect to this endpoint.
- **tunnel.cidr**: The CIDR block that defines the IP address range for cross-cluster communication.

To uninstall the chart:

```bash
helm delete fleetboard -n fleetboard-system
```

### Install/Uninstall FleetBoard on Child Cluster

To set up FleetBoard on a **Child Cluster** for cross-cluster communication, install the `fleetboard-agent` chart:

```bash
helm install fleetboard-agent fleetboard/fleetboard-agent --namespace fleetboard-system --create-namespace \
--set hub.hubURL=https://<Hub Public IP>:6443 --set cluster.clusterID=<Cluster Alias Name>
```

- **hub.hubURL**: The full URL (with port `6443`) of the Kubernetes API endpoint of the **Hub Cluster**. This allows the child cluster to register with the hub.
- **cluster.clusterID**: A unique alias for the child cluster to identify itself when communicating with the hub.

To uninstall the chart:

```bash
helm delete fleetboard-agent -n fleetboard-system
```

### Additional Setup Steps for FleetBoard

1. **Secrets Setup for Authentication**:
   - **Bootstrap Token**: The child cluster needs to use a valid **bootstrap token** to establish its first connection to the hub. You will need to create a Kubernetes secret that includes the **token ID** and **token secret** from the bootstrap token:
   
     ```bash
     kubectl create secret generic fleetboard \
       --from-literal=token-id=<Token ID> \
       --from-literal=token-secret=<Token Secret> \
       -n fleetboard-system
     ```

   - Ensure the secret is annotated correctly for usage in FleetBoard. Here’s an example of annotations:
   
     ```bash
     kubectl annotate secret fleetboard \
       auth-extra-groups=system:bootstrappers:fleetboard:register-token \
       usage-bootstrap-authentication=true \
       usage-bootstrap-signing=true \
       -n fleetboard-system
     ```

   - This secret will be referenced by the `cnf` pod on the child cluster to establish a secure connection with the hub cluster.

2. **DNS Configuration for Cross-Cluster Service Discovery**:
   - For DNS forwarding, you need to configure the child cluster's **CoreDNS** to forward DNS queries related to cross-cluster services to the **crossdns service** running on the child cluster. Add the following block to your CoreDNS configuration on the child cluster:
   
     ```yaml
     hyperos.local:53 {
         forward . <CrossDNS IP>
     }
     ```

   - **Important**: Do not add this DNS forwarding block to the **parent cluster**. It should only be configured in child clusters.

3. **Networking Setup**:
   - FleetBoard uses **WireGuard** tunnels for secure, encrypted communication between clusters. Ensure that **network policies** and **firewall rules** between the clusters allow traffic on the WireGuard port (`31820` by default) and that the child clusters can reach the hub’s public IP.

4. **Troubleshooting Tips**:
   - **Missing Secret**: If you encounter errors related to missing secrets (e.g., `get hub kubeconfig failed`), ensure that the secret containing the hub connection information (kubeconfig or bootstrap token) is correctly created in the `fleetboard-system` namespace.
   - **Logs**: Check the logs of the `cnf` and `controller` containers for errors related to network tunnels, DNS, or authentication:
   
     ```bash
     kubectl logs <pod-name> -n fleetboard-system -c cnf
     kubectl logs <pod-name> -n fleetboard-system -c controller
     ```

### Key Components and Flags

- **Hub Cluster**: The hub manages the tunnels and service synchronization between child clusters.
- **Child Cluster**: The child cluster connects to the hub using the `cnf` pod and establishes a secure VPN tunnel.
- **`--hub-secret-name`** and **`--hub-secret-namespace`**: These flags must point to the secret that holds the authentication token or kubeconfig for connecting the child to the hub.
- **WireGuard Tunnel**: FleetBoard sets up encrypted tunnels using WireGuard. Make sure the WireGuard kernel module is supported and enabled on your nodes.
