# Installing the logic apps
The logic apps use API connections which are authorised against other services - key vault and blob storage. This makes it difficult to deploy the logic apps directly from an ARM template export. See [here](https://www.feval.ca/posts/logicapp-ci/).

## Deployment Sequence
### Provision the other infastructure
Either manually of from the main template deploy:
1. Create a resource group 

### Create the logic app
Using the CLI and choosing your resource group (in this case *image-rg*):
```
az group deployment create -g "image-rg"   --template-file imagerecognition.json  --name "imagerecognition"   --query 'properties.provisioningState' -o tsv
```
Or in the portal inside the *image-rg* resource group hit *+Add* and choose "Template Deployment (Deplpoy using custom templates)", then choose "build your own template in the editor" and then paste into the template box the contents of *imagerecognition.json". 

Note that the Cognitive account, the storage account and key vault names need to be unique, so that you will need to amend these. For key vault provisioning, there is a requirement to use the *objectId* of your username that you sign into the Azure portal. This can be extracted from the portal and needs to be updated in the parameters of the template.

### Amend the logic app connections
The two logic apps shoould be deployed. The actions in the logic apps that use connections will need opening and the connections fixing. 

Once they have their connections amended the logic apps should save correctly and can then be tested.

### the find-the-celeb logic app
This is included in the main template. It needs amending in the same way as the image recognition logic app

### HTML pages
The simplest mechanism is to use blob storage for web site hosting.

The HTML files have a URL to the logic app POST endpoint - these need to be amended to work correctly
