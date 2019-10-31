# Installing the logic apps
The logic apps use API connections which are authorised against other services - key vault and blob storage. This makes it difficult to deploy the logic apps directly from an ARM template export. See [here](https://www.feval.ca/posts/logicapp-ci/).

## Deployment Sequence
### Provision the other infastructure
Either manually of from the main template deploy:
1. Create a resource group 
2. Cognitive Services Account
2. Key vault
3. Storage account
Elsewhere in these documents further configuration is described.

### Create the logic app
Using the CLI and choosing your resource group (in this case *image-reg*):
```
az group deployment create -g "image-rg"   --template-file image.json  --name "image"   --query 'properties.provisioningState' -o tsv
```

### Amend the logic app connections
The two logic apps shoould be deployed. The actions in the logic apps that use connections will need opening and the connections fixing. 

Once they have their connections amended the logic apps should save correctly and can then be tested.

### the find-the-celeb logic app
This is included in the main template. It needs amending in the same way as the image recognition logic app

### HTML pages
The simplest mechanism is to use blob storage for web site hosting.

The HTML files have a URL to the logic app POST endpoint - these need to be amended to work correctly
