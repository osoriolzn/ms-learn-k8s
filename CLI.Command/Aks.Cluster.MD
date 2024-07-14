# Create an Azure Kubernetes Service cluster

1. Sign in to Azure Cloud Shell with the account you want to deploy resources into.

    ℹ️ **Important:**

    We'll run all the scripts with Bash, so if you haven't created a Cloud Shell yet, select Bash as the running shell.

2. Create variables for the configuration values you reuse throughout the exercises.

    ```bash
    RESOURCE_GROUP=rg-contoso-video
    CLUSTER_NAME=aks-contoso-video
    LOCATION=eastus
    ```

3. Run the `az group create` command to create a resource group. Deploy all resources into this new resource group.

    ```bash
    az group create \
        --name $RESOURCE_GROUP \
        --location $LOCATION
    ```

4. Run the `az aks create` command to create an AKS cluster.

    To get the Kubernetes version, run the `az aks get-versions` command. The following query returns a non-preview Kubernetes version. Store that value in a Bash variable named `VERSION`. To retrieve and store the version number, run the following command:

    ```bash
    VERSION=$(az aks get-versions \
        --location $REGION_NAME \
        --query "values[?isPreview==null].version | [-1]" \
        --output tsv)
    ```

    ```bash
    <!-- Linux -->
    az aks create \
        --resource-group $RESOURCE_GROUP \
        --name $CLUSTER_NAME \
        --location $LOCATION \
        --kubernetes-version $VERSION \
        --node-count 2 \
        --load-balancer-sku standard \
        --vm-set-type VirtualMachineScaleSets \
        --node-vm-size Standard_DS2_v2 \
        --generate-ssh-keys \
        --network-plugin azure
    ```

    ```bash
    <!-- Windows -->
    az aks create \
        --resource-group $RESOURCE_GROUP \
        --name $CLUSTER_NAME \
        --location $LOCATION \
        --kubernetes-version $VERSION \
        --node-count 2 \
        --load-balancer-sku standard \
        --vm-set-type VirtualMachineScaleSets \
        --node-vm-size Standard_DS2_v2 \
        --generate-ssh-keys \
        --network-plugin azure \
        --windows-admin-username localadmin
    ```
    The `--windows-admin-username` parameter is used to set up administrator credentials for Windows containers, and prompts the user to set a password at the command line. The password has to meet [Windows Server password requirements](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference).

5. Run the `az aks nodepool add` command to add another node pool that uses the default Linux operating system.

    ```bash
    <!-- Linux -->
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name userpool \
        --node-count 2 \
        --node-vm-size Standard_DS2_v2
    ```

    ```bash
    <!-- Windows -->
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name uspool \
        --node-count 2 \
        --node-vm-size Standard_DS2_v2 \
        --os-type Windows
    ```
    The `--os-type` parameter is used to specify operating system of the node pool. If not specified, the command uses Linux as operating system for the nodes.

    ## Add a spot node pool to an AKS cluster

    A spot node pool can't be a system node pool for an AKS cluster. First, you need to create your cluster and then use the `az aks nodepool add` command to add a new user node pool.

    You set several parameters for a new node pool to configure it as a spot node pool.

    ```bash
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name spotpool01 \
        --enable-cluster-autoscaler \
        --max-count 3 \
        --min-count 1 \
        --priority Spot \
        --eviction-policy Delete \
        --spot-max-price -1 \
        --no-wait
    ```

# Link with kubectl

1. Link your Kubernetes cluster with `kubectl` by running the following command in Cloud Shell.

    ```bash
    az aks get-credentials \
        --name $CLUSTER_NAME \
        --resource-group $RESOURCE_GROUP
    ```
    This command adds an entry to your ~/.kube/config file, which holds all the information to access your clusters. Kubectl enables you to manage multiple clusters from a single command-line interface.

2. Run the kubectl get nodes command to check that you can connect to your cluster, and confirm its configuration.

    ```bash
    kubectl get nodes
    ```

3. Delete your current Kubernetes cluster with `kubectl` by running the following command in Cloud Shell.

    ```bash
    kubectl config delete-context $CLUSTER_NAME
    ```

# How to scale a node pool manually

```bash
az aks nodepool scale \
    --resource-group resourceGroup \
    --cluster-name aksCluster \
    --name nodePoolName \
    --node-count 0
```
