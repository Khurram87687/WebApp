# Tailwind Traders Backend Reference App

   ![](images/Diagram.png)


## Configure Build(CI) pipeline for Backend

1. Navigate to **Pipelines –> Builds**. Select **Backend-CI** and click **Edit**.
      
      ![](images/backend-ci-edit.png)

1. Your build pipeline will look like as below. Using this pipeline we are creating following required Azure resources for the application deployment
   - Azure Container Registry to store images
   - Azure Kubernetes cluster to deploy backend services
   - Azure SQL Database 
   - Cosmos DBs
   - Functions app service

   Then we will build the backend services as Docker images and push the images to ACR provisioned.

     ![](images/backend-ci-view.png)

1.  Select [Azure Resource Group Deployment](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceGroupDeploymentV2/README.md) task.
This task is used to create or update a resource group in Azure using the [Azure Resource Manager templates](https://azure.microsoft.com/en-in/documentation/articles/resource-group-template-deploy/). To deploy to Azure, an Azure subscription has to be linked to Azure Pipelines. Select your **Azure subscription** from Azure subscription dropdown. Click **Authorize**. If your subscription is not listed or to specify an existing service principal, follow the [Service Principal creation](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=vsts) instructions.

    ![](images/backend-ci-arm.png)

1. Select [ARM Outputs](https://github.com/keesschollaart81/vsts-arm-outputs) task.  This task enables you to use the ARM Deployment outputs in your Azure Pipelines. Select your **Azure subscription** from Azure subscription dropdown.

   ![](images/backend-ci-armoutputs.png)

1. Select **Azure CLI** task and select  **Azure subscription** from Azure subscription dropdown. Here we are using this task to set ACR (which is provisioned above using ARM task) name as variable 'registry' value.
    
    ![](images/backend-ci-azurecli.png)

1. Select **Build services** task. Here we are using [Docker task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=vsts) to build and push services images. Select your **Azure subscription** from Azure subscription dropdown and do the same for **Push services** task as well.

   ![](images/backend-ci-buildimages.png)

1. Select **[Publish Artifacts](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/publish-build-artifacts?view=azdevops)** task. Using this task we are publishing **Helm** scripts from source repository as build artifact to utilize the scripts in the release pipeline.

1. Select **Variables**. In this section, we have defined the Azure resource group name, sql admin login name, sql password and other required parameters. You have to enter your Azure service principal Client Id and Client Secret as shown below. If you don't have Azure service principal details create one by following the [Service Principal creation](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=vsts) instructions.

      ![](images/backend-ci-variables.png)

   > leave **registry** variable value blank only. The registry value will be set during the build dynamically.
1. Now **Save** the changes and **Queue** the build. All the tasks in the pipeline will be executed sequentially and you can see the progress of the build in the live console.
  
    ![](images/buildprogress.gif)

## Configuring services
 Before deploying services using **Helm** in Azure pipelines, you need to setup the configuration by editing the file `helm/gvalues.yaml` and put the secrets, connection strings and all the required configurations.

1. Navigate to **Repos**. Switch to the repo **TailwindTraders-Backend**

   ![](images/switch-repo-backend.png)

1. Open the `gvalues.yaml` file from the path `Deploy/helm` and click **Edit** to check out the file.

    ![](images/edit-gvalues.png)

   You need to edit this file and enter the required configuration parameters. 
   >Please refer to the comments of the file for its usage.

1. Navigate to Azure portal and select resource group **TailwindTradersBackend** which is provisioned using CI pipeline in the previous exercise. All the resources required to fill in the `gvalues.yaml` file you can get from the resources in the resource group **TailwindTradersBackend**. 

   ![](images/deployed-resources.png)

   Once you enter all the details the `gvalues.yaml` file will look like as below

   ```yaml
   # Global values for all helm charts. See comments for usage

   inf:
   sa: ttsa
   db:
    products:
      host: ttsqlserverwuh4r5fleek4q.database.windows.net
      port: "1433"
      catalog: Products
      user: sqladmin
      pwd: P2ssw0rd@123
    profile:
      host: ttsqlserverwuh4r5fleek4q.database.windows.net
      port: "1433"
      catalog: Profiles
      user: sqladmin
      pwd: P2ssw0rd@123
    coupons:
      user: ttcuoponsdbwuh4r5fleek4q
      pwd: fVL9OkerlpQgXzHqaBk1z6fBF7qyB6T99MKR2zmKeerMdhfIj4sKir86wZfm83PITsngRWswckM5fjn5cOKPMw==
      host: ttcuoponsdbwuh4r5fleek4q.documents.azure.com
      port: "10255"
      dbName: Coupon
      constr: mongodb://ttcuoponsdbwuh4r5fleek4q:fVL9OkerlpQgXzHqaBk1z6fBF7qyB6T99MKR2zmKeerMdhfIj4sKir86wZfm83PITsngRWswckM5fjn5cOKPMw==@ttcuoponsdbwuh4r5fleek4q.documents.azure.com:10255/?ssl=true&replicaSet=globaldb
      collection: CouponCollection   
    popularproducts:
      host: ttsqlserverwuh4r5fleek4q.database.windows.net
      port: "1433"
      catalog: PopularProducts
      user: sqladmin
      pwd: P2ssw0rd@123
    cart:
      host: https://ttshoppingdbwuh4r5fleek4q.documents.azure.com:443/
      auth: INmL9Uh1Pjp13jMrVrthcI3NgiaQ3lgibvwObl0uvntwlrGCQ9BGJwhu9fXGfWuurSotFgrE1ySGuFyDcczO5w==
   storage:
    productimages: https://ttstoragewuh4r5fleek4q.blob.core.windows.net/product-list
    productdetailimages: https://ttstoragewuh4r5fleek4q.blob.core.windows.net/product-detail
    couponimage: https://ttstoragewuh4r5fleek4q.blob.core.windows.net/coupon-list
    profileimages: https://ttstoragewuh4r5fleek4q.blob.core.windows.net/profiles-list
   appinsights:
    id: ""
   ingress:
    products:
      path: /product-api
    profile:
      path: /profile-api
    coupons:
      path: /coupons-api
    popularproducts:
      path: /popular-products-api
    stock:
      path: /stock-api
    imageclassifier:
      path: /image-classifier-api
    mobilebff:
      path: /mobilebff
    webbff:
      path: /webbff    
    cart:
      path: /cart-api
   apiurls:
    popularproductsapiurl: http://popularproducts
    productsapiurl: http://product 
    profileapiurl: http://profile
    couponsapiurl: http://coupons
    stockapiurl: http://stock
    imageclassifierapiurl: http://imageclassifier

   az:
   productvisitsurl: ""

   # Shared ingress configurations
   ingress:
   enabled: true
   annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
    ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/rewrite-target: /
   #  hosts:
   #    - <guid>.<region>.aksapp.io   # NOT NEEDED. SET BY SCRIPTS

   tls: []
   #  - secretName: chart-example-tls
   #    hosts:
   #      - chart-example.local

   ```

1. Once you are done with your changes **Commit** the changes.

   ![](images/commit-gvalues.png)

   Then navigate to **Pipelines --> Builds**, select **Backend-CI** and click **Queue** to trigger the new build so that we can get latest **helm** deployment files as artifacts in the build output. 

   ![](images/queue-build2.png)

1. Make sure you have Helm scripts folder in the build artifacts once the build is success.

     ![](images/helm-artifacts.png)

##  Configure Release(CD) pipeline for Backend

Now **Tailwind Traders** backend api's are published as Docker images and stored in ACR (a private Docker Registry hosted in Azure). We need to deploy all these images to **AKS** using helm charts. [Helm](https://helm.sh/) is a package manager for Kubernetes. Helm charts help you define, install, and upgrade even the most complex Kubernetes application.  Let us configure  **Release(CD) pipeline** for the same.

1. Navigate to **Pipelines --> Releases**. Select **Backend-CD** pipeline and click **Edit**.

    ![](images/backend-cd-edit.png)

1. Select **Dev** stage and click **View stage tasks** to view the pipeline tasks.

   ![](images/backend-cd-view.png)

   You will see the tasks as below.

    ![](images/backend-cd-tasks.png)

1. Select **Variables**. You need to enter your resources values in the variable values.
     ![](images/backend-cd-variables.png)
     It will look like as below once you fill all the details
     ![](images/backend-cd-variables2.png)

1. Switch to **Tasks** and select **[Helm install](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/helm-installer?view=azdevops)** task. this task is used to install **Helm** on agent machine. This is required to run Helm tasks.

   ![](images/backend-cd-helminstall.png)

1. Select **Azure CLI to get AKS credentials** task. Here we are using Azure CLI command to download the configuration files that **kubectl** needs to connect to your AKS cluster. Select your Azure subscription service connection.

   ![](images/backend-cd-azurecli1.png)

1. Select **Bash Script to install tiller** task. Helm is composed of two tools, one client-side (the Helm client) that needs to be installed on your machine(here it is agent machine), and a server component called **Tiller** that has to be installed on the Kubernetes cluster.  For deploying **Tiller** we are running the add-tiller.sh using Bash task.
   
    ![](images/backend-cd-tiller.png)

1. Select **Azure CLI to Create secrets on the AKS** task. Before deploying anything on AKS, a secret must be installed to allow AKS to connect to the ACR through a Kubernetes service account. To do so we are using the `./Create-Secret.sh` bash script with the following parameters:

   ![](images/backend-cd-azurecli2.png)

1. Remaining all tasks are **Helm** tasks to deploy the backend services images from ACR to AKS cluster. Select your Azure service connection details in all the tasks.

   ![](images/backend-cd-helmtasks.png)

   For more information about Helm task click [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/helm-deploy?view=azdevops)

1. Once you are done with your changes **Save** the pipeline and trigger the Release to deploy services to AKS cluster.

   ![](images/backend-cd-progress.gif)

1. Once the release is success run following commands in Azure cloud shell to view the deployed helm charts

   `az aks get-credentials -n <your-aks-name> -g <resource-group-name>`

   `helm ls`

    ![](images/helm-ls.png)

   Now the backend services required for Tailwind Traders application are deployed. To setup the Website (front-end) for Tailwind Traders follow the instructions in TailwindTraders Website wiki document.