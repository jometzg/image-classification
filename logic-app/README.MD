# Logic App Details
![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/logic-app.png "logic app")

There are three secrets in key vault
![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/key-vault-secrets.png "Key vault secrets")
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

# Find The Celebrity Logic App
This is very similar to the image classification logic app, but it has been extended to parse the response JSON and to return a different response string depending on whether the Cognitive Services call returns a *confidence* value of greater than 0.90.

The Logic app designer view is:

![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/celeb-logic-app.png "find the celeb logic app")

As can be seen from above, the HTTP response is parsed and one value tested (the *confidence*). It then sets the *reply* variable with a different response string before returning the response.

The logic app returns the bare miniumum text. In a real application, this would return a full HTML page, rather than a fragment.
