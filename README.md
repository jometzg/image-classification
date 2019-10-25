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

As can be seen from the diagram, there is much use of variables. This is purely to aid debugging. These steps could be removed and the expressions injected directly into each step.

# Demonstration front end
This is a really simple HTML page hosted in an app service. This has no direct active content, so could even be hosted elsewhere, such as blob storage. At the page's heart is:

```html
<form method="post" enctype="multipart/form-data" action="<logic app HTTP trigger URL>">
     <div>
    <label for="file">Choose file to upload</label>
    <input type="file" id="file" name="file" multiple>
    </div>
    <div>
    <button>Submit</button>
    </div>
</form>
```
![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/front-end.png "Simple demo front end")

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/front-end-select-image.png "Select and image file to upload")

For an image like this:

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/beech.png "London")

The response is JSON
>
{
	"description": {
		"tags": [
			"outdoor",
			"water",
			"nature",
			"beach",
			"rock",
			"sitting",
			"ocean",
			"mountain",
			"man",
			"standing",
			"people",
			"umbrella",
			"holding",
			"body",
			"boat",
			"rocky",
			"sand",
			"large",
			"wave",
			"bird",
			"river",
			"flying",
			"playing",
			"frisbee"
		],
		"captions": [
			{
				"text": "a body of water",
				"confidence": 0.9388402258579938
			}
		]
	},
	"requestId": "15d809cd-5a4d-42a6-bb5c-bcbc5f3ca169",
	"metadata": {
		"width": 1220,
		"height": 915,
		"format": "Jpeg"
	}
}
