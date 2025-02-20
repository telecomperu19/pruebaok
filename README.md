# Build a PHP and MySQL - Flexible Server app on Azure App Service

Build and deploy a sample PHP application to Azure App Service, and integrate it with Azure Database for MySQL - Flexible Server on the back end.

## STEP 1 - Create an Azure Database for MySQL flexible server

First, we'll provision a MySQL flexible server with public access connectivity, configure firewall rules to allow the application to access the server, and create a production database.

1. To create a MySQL flexible server, run the following command.

    **Note**: Replace your values for server name, admin username and password.

    ```azurecli-interactive
    az mysql flexible-server create \
    --name <your-mysql-server-name> \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --location southcentralus \
    --admin-user <your-mysql-admin-username> \
    --admin-password <your-mysql-admin-password> 
    ```

    You've now created a flexible server in the South Central US region. The server is based on the Burstable B1MS compute SKU, with 32 GB storage, a 7 day backup retention period, and configured with public access connectivity.

1. Next, to create a firewall rule for your MySQL flexible server to allow client connections, run the following command.

    **Note:** When both starting IP and end IP are set to 0.0.0.0, only other Azure resources (like App Services apps, VMs, AKS cluster, etc.) can connect to the flexible server.

    ```azurecli-interactive
    az mysql flexible-server firewall-rule create \
    --name <your-mysql-server-name> \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --rule-name AllowAzureIPs \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 0.0.0.0
    ```

1. To create a new MySQL production database *sampledb* to use with the PHP application, run the following command:

    ```azurecli-interactive
    az mysql flexible-server db create \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --server-name <your-mysql-server-name> \
    --database-name sampledb
    ```

## STEP 2 - Build your application

For the purposes of this exercise, we'll use a sample PHP application that displays and manages a product catalog. The application provides basic functionalities like viewing the products in the catalog, adding new products, updating existing item prices and removing products.

We'll directly clone the coded app and learn how to deploy it on Azure App Service.

**Note:** To learn more about the application code, go ahead and explore the app in the GitHub repository!

1. To clone the sample application repository and change to the repository root, run the following commands:

    ```azurecli-interactive
    git clone https://github.com/Azure-samples/php-mysql-app-service.git
    cd php-mysql-app-service
    ```

1. Run the following command to ensure that the default branch is `main`.

    ```azurecli-interactive
    git branch -m main
    ```

## STEP 3 - Create and configure an Azure App Service Web App

In Azure App Service (Web Apps, API Apps, or Mobile Apps), an app always runs in an App Service plan. An App Service plan defines a set of compute resources for a web app to run. In this step, we'll create an Azure App Service plan and an App Service web app within it, which will host the sample application.

1. To create an App Service plan in the Free pricing tier, run the following command:

    ```azurecli-interactive
    az appservice plan create --name plan-learn \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --sku FREE --is-linux
    ```

1. If you want to deploy an application to Azure web app using deployment methods like FTP or Local Git, you need to configure a deployment user with username and password credentials. After you configure your deployment user, you can take advantage of it for all your Azure App Service deployments.

    ```azurecli-interactive
    az webapp deployment user set \
    --user-name <your-deployment-username> \
    --password <your-deployment-password>
    ```

1. To create an App Service web app with PHP 8.0 runtime and to configure  the Local Git deployment option to deploy your app from a Git repository on your local computer, run the following command.

    **Note**: Replace `<your-app-name>` with a globally unique app name (valid characters are a-z, 0-9, and -).

    ```azurecli-interactive
    az webapp create \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --plan plan-learn \
    --name <your-app-name> \
    --runtime "PHP|8.0" \
    --deployment-local-git
    ```

    > [!IMPORTANT]
    In the Azure CLI output, the URL of the Git remote is displayed in the deploymentLocalGitUrl property, with the format `https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Save this URL, as you'll need it later.

1. Next we'll configure the MySQL flexible server database connection settings on the Web App.

    The `config.php` file in the sample PHP application retrieves the database connection information (server name, database name, server username and password) from environment variables using the `getenv()` function. In App Service, to set environment variables as **Application Settings** (*appsettings*), run the following command:

    ```azurecli-interactive
    az webapp config appsettings set \
    --name <your-app-name> \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --settings DB_HOST="<your-server-name>.mysql.database.azure.com" \
    DB_DATABASE="sampledb" \
    DB_USERNAME="<your-mysql-admin-username>" \
    DB_PASSWORD="<your-mysql-admin-password>" \
    MYSQL_SSL="true"
    ```

## STEP 4 - Deploy your application using Local Git

Now, we'll deploy the sample PHP application to Azure App Service using the Local Git deployment option.

1. Since you're deploying the main branch, you need to set the default deployment branch for your App Service app to main. To set the DEPLOYMENT_BRANCH under **Application Settings**, run the following command:

    ```azurecli-interactive
    az webapp config appsettings set \
    --name <your-app-name> \
    --resource-group <rgn>[Sandbox resource group name]</rgn> \
    --settings DEPLOYMENT_BRANCH='main'
    ```

1. Verify that you are in the application repository's root directory.

1. To add an Azure remote to your local Git repository, run the following command.

    **Note:** Replace `<deploymentLocalGitUrl>` with the URL of the Git remote that you saved in the **Create an App Service web app** step.

    ```azurecli-interactive
    git remote add azure <deploymentLocalGitUrl>
    ```

1. To deploy your app by performing a `git push` to the Azure remote, run the following command. When Git Credential Manager prompts you for credentials, enter the deployment credentials that you created in **Configure a deployment user** step.

    ```azurecli-interactive
    git push azure main
    ```

The deployment may take a few minutes to succeed.

## STEP 5 - Test your application

Finally, test the application by browsing to `https://<app-name>.azurewebsites.net`, and then add, view, update or delete items from the Product catalogue.

Congratulations! You have successfully deployed a sample PHP application to Azure App Service and integrated it with Azure Database for MySQL - Flexible Server on the back end.
