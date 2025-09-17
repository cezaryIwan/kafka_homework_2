## Infrastructure setup 
You can also refer to instructions from https://git.epam.com/epmc-bdcc/trainings/bd201/m11_kafkaconnect_json_azure <br>
### Create resources on Azure
```
az group create --name <RESOURCE_GROUP_NAME> --location <AZURE_REGION>
az storage account create --name <STORAGE_ACCOUNT_NAME> --resource-group <RESOURCE_GROUP_NAME> --location <AZURE_REGION> --sku Standard_LRS
az storage container create --name <CONTAINER_NAME> --account-name <STORAGE_ACCOUNT_NAME>
```
### Update configuration in main.tf file
### Deploy infrastructure with terraform
```
cd terraform
terraform init
terraform plan -out terraform.plan
terraform apply terraform.plan
```
### Set kubeconfig.yaml as Default for kubectl in Current Terminal Session
```
az aks get-credentials --resource-group <RESOURCE_GROUP_NAME_CREATED_BY_TERRAFORM> --name <AKS_NAME>
```
### Switch to the project kubernetes namespace
```
kubectl config set-context --current --namespace confluent
```
### Install Confluent for Kubernetes
```
helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes
```
### Configure and authenticate with ACR
```
az acr login --name <ACR_NAME>
```
### Build and push container using provisioned Dockerfile from "connectors" directory
```
docker build -t <ACR_NAME>/azure-connector:latest .
docker push <ACR_NAME>/azure-connector
az acr repository list --name <ACR_NAME> --output table
```
### Modify the file confluent-platform.yaml and replace the placeholder with ACR name
### Install all Confluent Platform components
```
kubectl apply -f confluent-platform.yaml
```
### Install a sample producer app and topic
```
kubectl apply -f producer-app-data.yaml
```
### Verify that all pods are running and in "Ready" status
```
kubectl get pods -o wide 
```
If your pods do not start correctly, e.g. they crash in a loop, try scaling down Zookeeper or Kafka pods. It's not recomended for production, but it's ok for this project on free cloud's tier. <br>
For me setup with 3 Kafka pods and only 1 Zookeper pod handled given exemplary data. <br>
### Set up port forwarding to Control Center web UI from local machine
```
kubectl port-forward controlcenter-0 9021:9021 &>/dev/null &
```
If you use virtual machine, remember to also forward ports to your physical machine! <br>
### Upload test data into "data" container in storage account, that was created via terraform in the previous steps
You can do it via azure portal platform, remember not to change directory structure <br>
### Prepare azure-source-cc.json, using example-azure-source-cc.json file
Update placeholders and add masking of data fields <br>
### Upload the connector file through the API
```
curl -s -X POST -H "Content-Type:application/json" --data @azure-source-cc.json http://localhost:8083/connectors
```
### Now you can verify data processing via Control Center http://localhost:9021
<img width="582" height="680" alt="image" src="https://github.com/user-attachments/assets/ea14eea8-310c-4924-ae15-7f2cfb6f9147" /> <br>
<img width="1386" height="930" alt="image" src="https://github.com/user-attachments/assets/4203bd57-26d6-40e7-a969-528c7237d7ff" /> <br>
Verify if date_time field is masked!
<img width="1435" height="788" alt="image" src="https://github.com/user-attachments/assets/76d78e09-c13b-4db4-81a8-0d91743198de" /> <br>
<img width="706" height="534" alt="image" src="https://github.com/user-attachments/assets/0d1d8f40-fc89-4d00-bd41-c2961f66e035" />






