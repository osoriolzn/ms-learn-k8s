# Create an Azure Kubernetes Service cluster

1. Sign in to Azure Cloud Shell with the account you want to deploy resources into.

    ℹ️ **Important:**

    We'll run all the scripts with Bash, so if you haven't created a Cloud Shell yet, select Bash as the running shell.

2. Create variables for the configuration values you reuse throughout the exercises.

    ```bash
    export RESOURCE_GROUP=rg-contoso-video
    export CLUSTER_NAME=aks-contoso-video
    export LOCATION=eastus
    ```

3. Run the `az group create` command to create a resource group. Deploy all resources into this new resource group.

    ```bash
    az group create --name=$RESOURCE_GROUP --location=$LOCATION
    ```

4. Run the `az aks create` command to create an AKS cluster.

    ```bash
    <!-- Linux -->
    az aks create \
        --resource-group $RESOURCE_GROUP \
        --name $CLUSTER_NAME \
        --node-count 2 \
        --generate-ssh-keys \
        --node-vm-size Standard_B2s \
        --network-plugin azure
    ```

    ```bash
    <!-- Windows -->
    az aks create \
        --resource-group $RESOURCE_GROUP \
        --name $CLUSTER_NAME \
        --node-count 2 \
        --generate-ssh-keys \
        --node-vm-size Standard_B2s \
        --network-plugin azure \
        --windows-admin-username localadmin
    ```
    The `--windows-admin-username` parameter is used to set up administrator credentials for Windows containers, and prompts the user to set a password at the command line. The password has to meet [Windows Server password requirements](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference).

5. Run the `az aks nodepool` add command to add another node pool that uses the default Linux operating system.

    ```bash
    <!-- Linux -->
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name userpool \
        --node-count 2 \
        --node-vm-size Standard_B2s
    ```

    ```bash
    <!-- Windows -->
    az aks nodepool add \
        --resource-group $RESOURCE_GROUP \
        --cluster-name $CLUSTER_NAME \
        --name userpool \
        --node-count 2 \
        --node-vm-size Standard_B2s \
        --os-type Windows
    ```
    The `--os-type` parameter is used to specify operating system of the node pool. If not specified, the command uses Linux as operating system for the nodes.

# Link with kubectl

1. Link your Kubernetes cluster with `kubectl` by running the following command in Cloud Shell.

    ```bash
    az aks get-credentials --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP
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