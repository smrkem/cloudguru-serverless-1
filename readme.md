## Serverless For Beginners - A Cloud Guru course project files

This is my working repo for the Serverless For Beginners course https://acloud.guru/learn/serverless-for-beginners  

### Video Transcoding Setup  
For the course I've set up 2 S3 buckets:
- ms-cloudguru-serverless-course-input
- ms-cloudguru-serverless-course-output  
and set the bucket policy on the 'output' bucket so anyone can retrieve files.

I set up an Elastic Transcoding pipeline for transcoding video uploads:
- cloudguru-serverless-pipeline

And then an event on the S3 input bucket to trigger the lambda function:
- cloudguru-serverless-transcode-video
which sends the input video through the pipeline into the output bucket.

### Website Setup
The code for the website is all provided in the course, the only difference being that I want to manually add the new parts from every chapter and at least attempt to understand what it's doing. Unfortunately the course doesn't bother to give even a short overview of the relevant javascript.  

### API Setup
The first thing the API is going to do is authenticate and fetch user info. I created the lambda function to get a user's profile from Auth0:  
- cloudguru-serverless-user-profile

and then set up the actual API in API Gateway. It's first resource is 'user-profile' with an appropriate GET method which gets proxied to the lambda function.
- cloudguru-serverless-serverlesstube  

The above API has the endpoint  
https://544hwejua6.execute-api.us-east-1.amazonaws.com/dev  

With all that in place, I updated the website to add some functionality to the user-profile button. Here's the commit:
https://github.com/smrkem/cloudguru-serverless-1/commit/ab94fe6c89579df67610a1a3531853dfd018644e  

The API is working and returning the user info fetched from Auth0 (in the user-profile lambda), but there's currently no authentication. We want to lock this functionality off to only authenticated users. We can create another lambda that compares the JWT sent in the request header with the client secret to authenticate it.

I add the lambda:  
- cloudguru-serverless-custom-authoriser  

And in API Gateway I add it to the Authorizers section and set up the GET to `user-profile` to use it.

(NOTE: In order for the custom authoriser to work I needed to change the algorithm in auth0 from 'RS256' to 'HS256'.)  

### Uploading a Video  
In order to upload videos directly to s3 in the javascript, we'll generaate a 'pre-signed url' for the input bucket and filename of the uploaded video. This will be retrieved after the user selects the file to upload, from a lambda function via API Gateway.  

To create the pre-signed URL, a user with upload permissions on the input bucket is needed with Access Key creds.  
- cloudguru-serverless-upload-s3  

The lambda function is then created setting the access keys, amongst other things, as environment variables.  
- cloudguru-serverless-get-upload-policy  

and a new resource `s3-policy-document` is added to the API with a GET method (which uses the same custom authoriser as before).  

It's also necessary to add a CORS configuration to the input bucket in S3 to allow for uploading from any domain.

Another big website update follows - here's the commit:  
https://github.com/smrkem/cloudguru-serverless-1/commit/8d4e85d4694262cf9075754e4e9f836e3f722b62

Loading up the website - everything is working. I can upload a video from the site, which ends up in the s3 input bucket, and the Elastic Transcoder role does its thing and eventually produces versions of the uploaded video in the output bucket.

woot!

There is an error however - subsequent requests to the API are getting 403'd - probably something to do with caching the token or on the custom authorizer. The TTL is set to 300s, but the generatePolicy on the custom-authoriser lambda is written to only allow permission for the requested resource. If, within the 300s a second request is made for a different resource, the API doesn't allow it.

The fix was to change the customAuthorizer code, hardcoding in the resource arns to allow invoking all the resources in the API, for now that's 2:
- GET .../user-profile
- GET .../s3-policy-document

everything is working now, including subsequent requests to the API.

### Adding Firebase to the App
Now to hook the front end up to a database. I created a firebase project called `cloudguru-serverless`
and imported the sample data provided.

Here's the final website update commit:
https://github.com/smrkem/cloudguru-serverless-1/commit/827f2d599dd88005c554d1454cd3be1b3e4f1e42  

and the site is now loading the sample videos which load and play fine. Firebase connection established! When I change the 'transcoding' value from 'false' to 'true' in firebase, the change is pushed and automatically changes on the website instantly.

In Firebase, I create a new Service Account for the app to use when updating the db, and download the access key json. Adding this to .gitignore so it stays private. I make the updates to the transcode-video lambda function, including the firebase access key json, and update my lambda function with the new package.
