# Image Classification Demonstration

This demonstration shows how Azure Cognitive Services REST API can be used to build a really simple image classification application.

## Component Parts
This is built on a few Azure resources:
1. Cognitive Services account
2. A logic app to process image classification requests
3. Blob storage to store images
4. A web app to display the user interface
5. Key vault for secrets

# Logic App
This is the heart of the demonstration.

![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/logic-app.png "Image classification flow")
In the above diagram, the logic app responds to HTTP Post requests. In this case the post request is not a JSON POST, but is a multipart/form-data one. This is the format a browser page uses when uploading a file. This allows the logic app to be directly called from the submit of a file upload web page.

The steps this logic app takes are:
1. Get secrets from key vault
2. Pull out the file name
3. Pull out the base64 encoded file data
4. Clreate a blob with the name and bease64 decoded file contents
5. Call the Cognitive services image classification endpoint with its key (from key vault)
6. Return its response as the response to the overall logic app call

# Demonstration front end


