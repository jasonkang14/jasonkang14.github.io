---
title: Django - Upload Image to AWS S3
date: "2019-08-07T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Django-upload-image-to-aws-s3"
category: "Django"
tags:
  - "Django"
  - "Cloud"
description: "How to upload an image to AWS S3 using Django"
---

After receiving an image file from a client, you have to use a server to upload the image to AWS S3. I am not going to post about how to set up IAM and S3 Bucket in AWS as I believe there are already so many posts about it.

If you want to upload an image to AWS S3, you need to first install `boto3`, which is a library that helps you upload a file to AWS S3.

`pip install boto3` <br>
and please build a habit of `pip freeze > requirements.txt` in order to help those who are working on the same project with you.

Check out the code below

```python
import boto3

...

class UploadPictureView(APIView):
    permission_classes = [IsAuthenticated]
    s3_client = boto3.client(
        's3',
        aws_access_key_id     = yptest_aws_access_key_id,
        aws_secret_access_key = yptest_aws_secret_access_key
    )

    def post(self, request):
        picture = request.FILES['image']
        picture.name = f"{request.user.id}_{picture.name}"

        self.s3_client.upload_fileobj(
            picture,
            "yptest-sns",
            picture.name,
        )

        return JsonResponse({
            "picture_uri": f"https://yptest-sns.s3.ap-northeast-2.amazonaws.com/{picture}"
        }, safe=False)
```

You can retrieve your `aws_access_key_id` and `aws_secret_access_key` from your `AWS IAM`. <br>

Once you upload an image to `AWS S3`, it provides you with an uri specific to the image by adding the image file name to the end of the `S3 bucket uri`. So if you return the uri to your client, the client will send a request to your server with the uri to save the information into your database
