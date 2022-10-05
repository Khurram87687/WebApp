# Tailwind Traders Website Reference App

   ![](images/website.png)


## Configure Build(CI) pipeline for Website

1. Navigate to **Pipelines –> Builds**. Select **Website-CI** and click **Edit**.

    ![](images/website-ci-edit.png)

1. Your build pipeline will look like as below. Using this pipeline we are creating following required Azure resources for the application deployment
    
      - Azure Container Registry
      - Web App for Containers

    Then we will build the application Docker image and push the image to ACR provisioned.

   ![](images/ci-definition.png)

1. Select [Azure Resource Group Deployment](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceGroupDeploymentV2/README.md) task.
This task is used to create or update a resource group in Azure using the [Azure Resource Manager templates](https://azure.microsoft.com/en-in/documentation/articles/resource-group-template-deploy/). To deploy to Azure, an Azure subscription has to be linked to Azure Pipelines. Select your **Azure subscription** from Azure subscription dropdown. Click **Authorize**. If your subscription is not listed or to specify an existing service principal, follow the [Service Principal creation](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=vsts) instructions.

    ![](images/ARMdeployment.png)

1. Select [ARM Outputs](https://github.com/keesschollaart81/vsts-arm-outputs) task.  This task enables you to use the ARM Deployment outputs in your Azure Pipelines. Select your **Azure subscription** from Azure subscription dropdown.

   ![](images/ARM-outputs.png)

1. Select **Build an Image** task. Here we are using [Docker task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/docker?view=vsts) to build and push application image. Select your **Azure subscription** from Azure subscription dropdown and do the same for **Push image** task as well.

    ![](images/build-image.png)

1. Select **Variables**. In this section, we have defined Azure resource group name, location and other required parameters for the build pipeline as variables.

      ![](images/variables.png)

1. Now **Save** the changes and **Queue** the build. All the tasks in the pipeline will be executed sequentially and you can see the progress of the build in the live console.
  
    ![](images/buildprogress.gif)

1. Once the build is success go to your Azure portal and navigate to resource group **TailwindTraderWeb**. You should be able to see the following resources which were deployed during the build.

   ![](images/azure-resources.png)

   And you should be able to see repository with name **website** in your Container registry.
  
   ![](images/acr-repository.png)


## Configure Build(CD) pipeline for Website

1. Navigate to **Pipeline » Releases**. Select **Website-CD** and click **Edit** pipeline.
   
    ![](images/edit-releasepipeline.png)

1. Select **Dev** stage and click **View stage tasks** to view the pipeline tasks.

    ![](images/viewstagetasks.png)

    You will see the tasks as below.

    ![](images/release-tasks.png)

1. Select **Variables**. You need to enter your ACR and App service (which are provisioned in build pipeline) details here. We will make use of these variable values in our pipeline tasks.
   
    ![](images/variables-release.png)

   To get the details navigate to the Azure resource group provisioned earlier.
   Make a note of App service name

    ![](images/appservice-name.png)

   Navigate to ACR and select **Access keys**. Make a note of the following details

   ![](images/acr-details.png)

   Enter the above values for the appropriate variables.

1. Select  **Azure CLI** task. Here we are using Azure CLI script to set the container settings for the Azure app service we created. Select the **Azure service connection** from the drop down. 
 
    ![](images/azurecli-task.png)

1. Select **App Service** task. Make sure you have selected **Azure service connection** and **App service name**.
     
     ![](images/appservice-task.png)

1. Select **[Azure App Service manage](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureAppServiceManageV0/README.md)** task. Select the required parameters as shown below. We are using this task here to restart the app service.
 
   ![](images/restart-appservice.png)

1. **Save** the changes and queue the release.

    ![](images/release-progress.gif)

1. Once the release is success navigate to your Azure portal. Select the app service that we created and browse to view the application deployed.

   ![](images/browse-appservice.png)

   ![](images/website-view.png)
