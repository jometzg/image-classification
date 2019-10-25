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

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/front-end-select-image.png "Seelct and image file to upload")

The response is JSON
>
{
	"description": {
		"tags": [
			"outdoor",
			"water",
			"building",
			"boat",
			"river",
			"bridge",
			"large",
			"front",
			"city",
			"riding",
			"train",
			"traveling",
			"clock",
			"old",
			"harbor",
			"man",
			"standing",
			"white",
			"track",
			"tower",
			"group"
		],
		"captions": [
			{
				"text": "a bridge over water with a city in the background",
				"confidence": 0.8862045336564989
			}
		]
	},
	"requestId": "a020bda5-c5c1-426e-a60e-20dad6f00fbb",
	"metadata": {
		"width": 640,
		"height": 360,
		"format": "Jpeg"
	}
}
