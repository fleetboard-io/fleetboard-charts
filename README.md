## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

    helm repo add fleetboard https://github.com/fleetboard-io/fleetboard-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
fleetboard` to see the charts.


### Install/Uninstall  Fleetboard on Hub Cluster
To install the `fleetboard` chart:

    helm install fleetboard fleetboard/fleetboard --namespace fleetboard-system  --create-namespace \
    --set tunnel.endpoint=<Hub Pubic IP>   --set tunnel.cidr=20.112.0.0/12

To uninstall the chart:

    helm delete fleetboard -n fleetboard-system 


### Install/Uninstall Fleetboard on Child Cluster

To install the `fleetboard-agent` chart:

    helm install fleetboard-agent fleetboard/fleetboard-agent --namespace fleetboard-system  --create-namespace \
    --set hub.hubURL=https://<Hub Public IP>:6443 --set cluster.clusterID=<Cluster Alias Name>

To uninstall the chart:

    helm delete fleetboard-agent -n fleetboard-system 
