# Image Classification Demonstration

This demonstration shows how Azure Cognitive Services REST API can be used to build a really simple image classification application.

## Component Parts
This is built on a few Azure resources:
1. Cognitive Services account
2. A logic app to process image classification requests
3. Blob storage to store images
4. A web app to display the user interface
5. Key vault for secrets

See here for more [details](https://github.com/jometzg/image-classification/tree/master/logic-app)

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

# Find the celebrity
This is a slightly different caspability, which attempts to name a person given their picture. This uses the same Cognitive Services account, but a slightly different API URL. It then returns some JSON with the name of the person and a level of confidence.

Most of this demonstration is identical to that of the image classification one, except:
1. the Cognitive services URL is slightly different
2. The response is parsed and returns a different response if the confidence is greater that 90% ( 0.90)

The Cognitive URL is of the form:
```
POST https://<region>.api.cognitive.microsoft.com/vision/v1.0/models/celebrities/analyze
{
  "url": "https://<blob-account>.blob.core.windows.net/images/jj.png"
}
```
The logic app for this is *find-the-celeb* and can be found in the "logic-app" folder. There is also a piece of HTML *celeb.htm* in the "user-interface" folder which can be either put in a web app or blob storage to generate the user interface. This HTML page only differs in its form *action* endpoint.

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/barak.jpg "Simple image")

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/front-end-select-image-barak.png "Select and image file to upload")

![alt text](https://github.com/jometzg/image-classification/blob/master/user-interface/front-end-celeb-response.png "Celebrity response")
