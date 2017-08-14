## Serverless For Beginners - A Cloud Guru course project files

This is my working repo for the Serverless For Beginners course https://acloud.guru/learn/serverless-for-beginners  

### S3 Setup  
For the course I've set up 2 S3 buckets:
- ms-cloudguru-serverless-course-input
- ms-cloudguru-serverless-course-output  
and set the bucket policy on the 'output' bucket so anyone can retrieve files.

I set up an Elastic Transcoding pipeline for transcoding video uploads:
- cloudguru-serverless-pipeline

And then an event on the S3 input bucket to trigger the lambda function:
- cloudguru-serverless-transcode-video
which sends the input video through the pipeline into the output bucket.
