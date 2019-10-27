# User Interface

The user interface is a plain old HTML file with a standard file upload capability and an HTML POST to the logic app.

As such, the HTML page can be hosted almost anywhere. For the purposes of this demonstration, there are 2 options:
1. Azure App Service (Web App)
2. Blob storage

## Web App
A web app may be created and the HTML files put into its "wwwroot" folder. This can be done through serveral mechanims:
1. CI/CD pipeline in Azure DevOps
2. Going into the web app Kudu console and creating a file and pasting the HTML
3. Using an FTP client to copy the file to "wwwroot"

Using Kudu is shown below:

![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/kudu.png "web app Kudu site hosting")



## Blob Storage
Blob storage can be [configured](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) to host static web sites. This is really efficient as you can re-use the storage account that stores the images.

The configuration setting for this can be found in the portal and this then results in a blob folder called *$web*:

![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/static-site-hosting-in-storage.png "static site hosting")

As can be seen from the above, screenshot, there is a *$web* container in addition to the *images* container that is used for the user uploaded images.

For both the image recognition and find a celebrity HTML pages, these can be in the root of the *$web* container:

![alt text](https://github.com/jometzg/image-classification/blob/master/logic-app/storage-hosted-html-pages.png "static site hosting")
