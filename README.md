## Usage

[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:

    helm repo add nauti https://nauti-io.github.io/helm-charts

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
nauti` to see the charts.


### Install/Uninstall  Nauti on Hub Cluster
To install the `nauti` chart:

    helm install nauti nauti/nauti --namespace nauti-system  --create-namespace \
    --set tunnel.endpoint=<Hub Pubic IP>   --set tunnel.cidr=20.112.0.0/12

To uninstall the chart:

    helm delete nauti -n nauti-system 


### Install/Uninstall Nauti on Child Cluster

To install the `nauti-agent` chart:

    helm install nauti-agent nauti/nauti-agent --namespace nauti-system  --create-namespace \
    --set hub.hubURL=https://<Hub Public IP>:6443 --set cluster.clusterID=<Cluster Alias Name>

To uninstall the chart:

    helm delete nauti-agent -n nauti-system 
