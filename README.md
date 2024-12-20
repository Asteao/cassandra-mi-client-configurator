# Azure Managed Instance for Apache Cassandra Client Configurator

The Azure  Client configurator is a tool designed to assist you in configuring a hybrid cluster and simplifying the migration process to [Azure Managed Instance for Apache Cassandra](https://learn.microsoft.com/azure/managed-instance-apache-cassandra/introduction). If you currently have on-premises datacenters or are operating in a self-hosted environment, you can use Azure Managed Instance for Apache Cassandra to seamlessly incorporate extra datacenters into your cluster while effectively maintaining them.

## Prerequisites

* Ensure that both the Azure Managed Instance and on-premises Cassandra cluster are located on the same virtual network. If not, it is necessary to establish network peering or other means of connectivity (for example, express route).
* The cluster name for both the Managed cluster and local cluster must be the same.
        * In the cassandra.yaml file ensure the storage port is set to 7001 and  the cluster name is same as the managed cluster:

 ```bash
cluster_name: managed_cluster-name
storage_port: 7001
 ```

```sql
UPDATE system.local SET cluster_name = 'managed_cluster-name' where key='local';
```

## Installation

* Download and navigate into the client configurator folder.
* Set up a virtual environment to run the python script:

```bash
python3 -m venv env
source env/bin/activate
python3 -m pip install -r requirements.txt
```

* Sign into Azure CLI `az login`
* Run the python script within the client folder with information from the existing (on-premises) cluster:

```python
python3 client_configurator.py --subscription-id <subcriptionId> --cluster-resource-group <clusterResourceGroup> --cluster-name <clusterName> --initial-password <initialPassword> --vnet-resource-group <vnetResourceGroup> --vnet-name <vnetName> --subnet-name <subnetName> --location <location> --seed-nodes <seed1 seed2 seed3> --mi-dc-name <managedInstanceDataCenterName> --dc-name <onPremDataCenterName> --sku <sku>
```

> [!NOTE]
>
> ```bash
> --seed-nodes, the seed nodes of the existing datacenters in your on-premises or self-hosted Cassandra cluster.
> --dc-name: The data center name of your Azure Managed Instance cluster.
> ```

* The Python script produces a tar archive named `install_certs.tar.gz`.
        * Unpack this folder into `/etc/cassandra/` on each node.

    ```bash
    sudo tar -xzvf install_certs.tar.gz -C /etc/cassandra
    ```

* Inside the directory from step 5, run `sudo ./install_certs.sh`.
        *Ensure that the script is executable by running `sudo chmod +x install_certs.sh`.
        *The script installs and point Cassandra towards the new certs needed to connect to the Azure Managed Instance cluster.
        *It then prompts user to restart Cassandra.
        :::image type="content" source="./media/script-result.png" alt-text="Screenshot of the result of running the script":::

* Once Cassandra has finished restarting on all nodes, check `nodetool status`. Both datacenters should appear in the list, with their nodes in the UN (Up/Normal) state.

## Next steps

To learn more about Azure Managed Instance for Apache Cassandra, please visit our [product page](https://learn.microsoft.com/azure/managed-instance-apache-cassandra/configure-hybrid-cluster) and [documentation](https://learn.microsoft.com/azure/managed-instance-apache-cassandra/).
