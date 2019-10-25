# Logic App Details

```json
{
    "$connections": {
        "value": {
            "azureblob": {
                "connectionId": "/subscriptions/2e500704-1ab3-49c6-bbbb-8b2bec98ffdc/resourceGroups/image-recognition-rg/providers/Microsoft.Web/connections/azureblob",
                "connectionName": "azureblob",
                "id": "/subscriptions/2e500704-1ab3-49c6-bbbb-8b2bec98ffdc/providers/Microsoft.Web/locations/westeurope/managedApis/azureblob"
            },
            "keyvault_1": {
                "connectionId": "/subscriptions/2e500704-1ab3-49c6-bbbb-8b2bec98ffdc/resourceGroups/image-recognition-rg/providers/Microsoft.Web/connections/keyvault-1",
                "connectionName": "keyvault-1",
                "id": "/subscriptions/2e500704-1ab3-49c6-bbbb-8b2bec98ffdc/providers/Microsoft.Web/locations/westeurope/managedApis/keyvault"
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
