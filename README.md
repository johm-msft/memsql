# memsql

Welcome to Version 1 of the MEMSQL ARM Tempalte for Azure.

This template allows you to deploy a complete memsql cluster into Azure, with control over the amount of leaf and aggregator nodes you wish to deploy. 

The template the following resources in order
1. Virtual Network with 3 Subnets
2. Availability Sets (1 for Leaf Nodes, 1 for Aggregator Nodes)
3. Aggregator and Leaf Nodes are deployed in Parrallel
4. Master Node with Public IP and Network Security Groups with Port 22 and 8080 open

With no changes to the template it will by default deploy a single master aggregator, 2 child aggregator and 2 leaf nodes into a memsql available set. 

On Steps 3 and 4, the nodes are configured using Cloud-Init which will configure the attached disks and get it ready for memsql. It will download and deploy the latest edition of memsql and form a cluster. There are contorl signals waiting for events to happen to allow the nodes catchup on each other. There are also a dependancy chain built into the deployment to ensure the master ndoe does not get deployed until after the aggregators and leaf nodes have been deployed.

#Important Before you deploy
There are a couple of things that need to be done before attempting to deploy to Azure

1. Check your qouta limits in Azure and ensure you have enough CPU resources available to deploy and select the correct size you want for the virtual machine. The default size in template is Standard_DS3_v2, which is enough to run the installer and test but not enough for actual production workloads.
2. Obtain a valid license from memsql, they are available from the customer portal (http://portal.memsql.com) please copy the entire license key into the parameters files
3. Generate a temporary ssh key and use it in the parameters files for the initial build.
4. Set a memsqlRootPassword in the Parameters files.

#Deploying to Azure

1. Open `https://portal.azure.com`
2. Open CloudShell in the top right hand corner look for the >_ symbol
3. Change to your cloud drive type `cd /usr/<username>/clouddrive`
4. Clone the repo using `git clone https://github.com/johm-msft/memsql.git`
5. Modify the azuredeploy.parameters.json file, target the sshKeyData, memsqlLicense and memsqlRootPassword, use nano to make and save the edits. For aware the sshKey data should include the `ssh-rsa <publickey>` format
6. Create a new resource group in the region you want to deploy for example `New-AzureRmResourceGroup -Name test -Location westus2`
7. Create a new resource group deployment using the resource group you just created `New-AzureRmResourceGroupDeployment -TemplateFile ./azuredeploy.json -TemplateParameterFile ./azuredeploy.parameters.json -ResourceGroupName test`
8. Deployment of the Infrastructure resources are pretty quick, however the full installation and cluster formation can take 15-30 minutes. The easiest way to check is to find the public ip address provisioned for your memsql master node and use a browser to https://<publicipofmaster>:8080 and when it displays the MemSql Studio webpage you will know it is active!

