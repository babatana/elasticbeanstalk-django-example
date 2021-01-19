Elastic Beanstalk Django Example Project
==================================================

This sample code helps get you started with a simple Django web application
deployed by AWS Elastic Beanstalk and the [Elastic Beanstalk CLI](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html).

What's Here
-----------

This sample includes:

* README.md - this file
* ebdjango/ - this directory contains your Django project files. Note that this
  directory contains a Django config file (settings.py) that includes a pre-defined
  SECRET_KEY. Before running in a production environment, you should replace this
  application key with one you generate
  (see https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/#secret-key for details)
* helloworld/ - this directory contains your Django application files
* manage.py - this Python script is used to start your Django web application
* .ebextensions/ - this directory contains the Django configuration file that
  allows AWS Elastic Beanstalk to deploy your Django application
* requirements.txt - this file is used to install Python dependencies needed by
  the Django application


## Static Files

This project also shows how to use S3 to host the static files.

### Prerequisites

1. Make sure the IAM role used by the Elastic Beanstalk environment has permissions to read/write to
the S3 bucket.

__example policy with cloudfront__
```json
{
    "Version": "2012-10-17",
    "Id": "Policy1610741923542",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E3FB3I52FIBSOV"
            },
            "Action": [
                "s3:GetObject*",
                "s3:GetBucket*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::BUCKET_NAME",
                "arn:aws:s3:::BUCKET_NAME/*"
            ]
        },
        {
            "Sid": "Stmt1610741920838",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111111111111:role/aws-elasticbeanstalk-ec2-role"
            },
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::BUCKET_NAME/*"
        },
        {
            "Sid": "Stmt1610741920838",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111111111111:role/aws-elasticbeanstalk-ec2-role"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::BUCKET_NAME"
        }
    ]
}
```

2. Update the [ebdjango/settings.py](./ebdjango/settings.py) file with the following settings:

```python
AWS_STORAGE_BUCKET_NAME = 'S3_BUCKET_NAME_HERE'
AWS_S3_REGION_NAME = 'us-east-2'
AWS_S3_CUSTOM_DOMAIN =  'CUSTOM_DOMAIN_HERE' # i.e. d2wk3ltjsmcl9j.cloudfront.net
STATICFILES_LOCATION = 'static'
STATICFILES_STORAGE = 'custom_storages.StaticStorage'
MEDIAFILES_LOCATION = 'media'
DEFAULT_FILE_STORAGE = 'custom_storages.MediaStorage'

INSTALLED_APPS = [
    ...
    'storages',
]
```

## Deployment

In order to deploy the app to Elastic Beanstalk we need to add a `predeploy` hook to run `collectstatic`.
This will also handle uploading all of our static files to S3.

You can view the script used at [.platform/hooks/predeploy/01_collectstatic.sh](./platform/hooks/predeploy/01_collectstatic.sh)
