# Logic App Details

There are three secrets in key vault
![alt text](https://github.com/jometzg/cognitive-speech/blob/master/logic-app/key-vault-secrets.png "Key vault secrets")
These are used to contruct the Cognitive services request.

The Cognitive services call is of the form:
```
POST https://westeurope.api.cognitive.microsoft.com/vision/v1.0/describe?maxCandidates=1 HTTP/1.1 
Host: westeurope.api.cognitive.microsoft.com 
Content-Type: application/json 
Ocp-Apim-Subscription-Key: XXXX-COGNITIVE-KEY-XXXX 
{
    "url":"< URL of the image put into blob storage>"
}
```
So, the HTTP trigger in the logic app takes the inbound file, stores this in a *public* blob container and then uses that as the URL to the JSON in the POST to the Cognitive Services image classification API.

The JSON version of the logic app code is below:
```json
{
    "$connections": {
        "value": {
            "azureblob": {
                "connectionId": "/subscriptions/XXXXXXXXXXXXXXXXXXXX/resourceGroups/image-recognition-rg/providers/Microsoft.Web/connections/azureblob",
                "connectionName": "azureblob",
                "id": "/subscriptions/2e500704-1ab3-49c6-bbbb-8b2bec98ffdc/providers/Microsoft.Web/locations/westeurope/managedApis/azureblob"
            },
            "keyvault_1": {
                "connectionId": "/subscriptions/XXXXXXXXXXXXXXXXXXXX/resourceGroups/image-recognition-rg/providers/Microsoft.Web/connections/keyvault-1",
                "connectionName": "keyvault-1",
                "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXX/providers/Microsoft.Web/locations/westeurope/managedApis/keyvault"
            }
        }
    },
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "Create_blob_2": {
                "inputs": {
                    "body": "@base64ToBinary(triggerBody()?['$multipart'][0]['body']['$content'])",
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['azureblob']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/datasets/default/files",
                    "queries": {
                        "folderPath": "/images",
                        "name": "@variables('filename')",
                        "queryParametersSingleEncoded": true
                    }
                },
                "runAfter": {
                    "Set_Filename": [
                        "Succeeded"
                    ]
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                },
                "type": "ApiConnection"
            },
            "Get_blob_url": {
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['keyvault_1']['connectionId']"
                        }
                    },
                    "method": "get",
                    "path": "/secrets/@{encodeURIComponent('blob-url')}/value"
                },
                "runAfter": {
                    "Get_cognitive_url": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
            "Get_cognitive_key": {
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['keyvault_1']['connectionId']"
                        }
                    },
                    "method": "get",
                    "path": "/secrets/@{encodeURIComponent('cognitive-key')}/value"
                },
                "runAfter": {},
                "type": "ApiConnection"
            },
            "Get_cognitive_url": {
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['keyvault_1']['connectionId']"
                        }
                    },
                    "method": "get",
                    "path": "/secrets/@{encodeURIComponent('congitive-url')}/value"
                },
                "runAfter": {
                    "Get_cognitive_key": [
                        "Succeeded"
                    ]
                },
                "type": "ApiConnection"
            },
            "HTTP": {
                "inputs": {
                    "body": {
                        "url": "@{body('Get_blob_url')?['value']}/images/@{variables('filename')}"
                    },
                    "headers": {
                        "Ocp-Apim-Subscription-Key": "@body('Get_cognitive_key')?['value']"
                    },
                    "method": "POST",
                    "queries": {
                        "maxCandidates": "1"
                    },
                    "uri": "@{body('Get_cognitive_url')?['value']}/vision/v1.0/describe"
                },
                "runAfter": {
                    "Create_blob_2": [
                        "Succeeded"
                    ]
                },
                "type": "Http"
            },
            "Initialize_filename": {
                "inputs": {
                    "variables": [
                        {
                            "name": "filename",
                            "type": "String",
                            "value": "xxx"
                        }
                    ]
                },
                "runAfter": {
                    "init_cd": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Response": {
                "inputs": {
                    "body": "@body('HTTP')",
                    "statusCode": "@outputs('HTTP')['statusCode']"
                },
                "kind": "Http",
                "runAfter": {
                    "HTTP": [
                        "Succeeded"
                    ]
                },
                "type": "Response"
            },
            "Set_CD": {
                "inputs": {
                    "name": "cd",
                    "value": "@{triggerBody()?['$multipart'][0]['headers']['Content-Disposition']}"
                },
                "runAfter": {
                    "init_length": [
                        "Succeeded"
                    ]
                },
                "type": "SetVariable"
            },
            "Set_Filename": {
                "inputs": {
                    "name": "filename",
                    "value": "@{substring(variables('cd'), 34, sub(variables('length'), 35))}"
                },
                "runAfter": {
                    "Set_length": [
                        "Succeeded"
                    ]
                },
                "type": "SetVariable"
            },
            "Set_length": {
                "inputs": {
                    "name": "length",
                    "value": "@length(variables('cd'))"
                },
                "runAfter": {
                    "Set_CD": [
                        "Succeeded"
                    ]
                },
                "type": "SetVariable"
            },
            "init_cd": {
                "inputs": {
                    "variables": [
                        {
                            "name": "cd",
                            "type": "String",
                            "value": "XXX"
                        }
                    ]
                },
                "runAfter": {
                    "Get_blob_url": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "init_length": {
                "inputs": {
                    "variables": [
                        {
                            "name": "length",
                            "type": "Integer",
                            "value": 0
                        }
                    ]
                },
                "runAfter": {
                    "Initialize_filename": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "manual": {
                "inputs": {
                    "schema": {
                        "properties": {
                            "$content": {
                                "type": "string"
                            },
                            "$content-type": {
                                "type": "string"
                            },
                            "$multipart": {
                                "items": {
                                    "properties": {
                                        "body": {
                                            "properties": {
                                                "$content": {
                                                    "type": "string"
                                                },
                                                "$content-type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        },
                                        "headers": {
                                            "properties": {
                                                "Content-Disposition": {
                                                    "type": "string"
                                                },
                                                "Content-Type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        }
                                    },
                                    "required": [
                                        "headers",
                                        "body"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "kind": "Http",
                "type": "Request"
            }
        }
    }
}
```

Note that the connections to blob storage and key vault will need to be recreated for your blob storage and key vault and the secrets need to be manually entered into key vault.

It should also be noted that a logic app creates its own POST URL on first Save. It is this that needs to be put in the HTML form as the POST action.

This can also be tested without the user interface with Postman.

```
POST <Logic app HTTP trigger URL> HTTP/1.1
Host: xxxhost.logic.azure.com:443
Ocp-Apim-Subscription-Key: XXXX-KEY-XXXXXXXXXXX
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
User-Agent: PostmanRuntime/7.17.1
Accept: */*
Cache-Control: no-cache
Postman-Token: b7cd145e-f58c-4456-b1f3-da78a3a2769c,98bffb80-fa4c-4f81-a2ef-43d9311394ed
Host: prod-87.westeurope.logic.azure.com:443
Accept-Encoding: gzip, deflate
Content-Length: 544910
Connection: keep-alive
cache-control: no-cache


Content-Disposition: form-data; name="file"; filename="/C:/Users/jometzg/Downloads/train-station.jpg


------WebKitFormBoundary7MA4YWxkTrZu0gW--,
Content-Disposition: form-data; name="file"; filename="/C:/Users/jometzg/Downloads/train-station.jpg


------WebKitFormBoundary7MA4YWxkTrZu0gW--
Content-Disposition: form-data; name="destination"

train-station.jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```
