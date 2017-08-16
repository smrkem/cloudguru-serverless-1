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
