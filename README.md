![Alturos.ImageAnnotation](doc/logo-banner.png)

# Alturos.ImageAnnotation

The purpose of this project is to manage training data for Neural Networks. The data is stored at a central location, for example Amazon S3.
In our case we have image data for different runs that we want to annotate together. Every run is stored in an AnnotationPackage.
For every AnnotationPackage you can set your own tags... this information is stored in the Amazon DynamoDB.

![object detection result](/doc/AlturosImageAnnotation.png)

## Features

 - Collaborative annotation of images
 - Verification of image annotation data
 - Export for yolo (train.txt, test.txt, obj.names) with filters
 - No requirement for a custom server

## Keyboard Shortcuts

Shortcut | Description | 
--- | --- |
<kbd>↓</kbd> | Next image |
<kbd>↑</kbd> | Previous image |
<kbd>→</kbd> | Next Object Class |
<kbd>←</kbd> | Previous Object Class |
<kbd>0</kbd>-<kbd>9</kbd> | Select Object Class |
<kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd><br>+<kbd>Shift</kbd><br>+<kbd>Ctrl</kbd><br>+<kbd>Alt</kbd> | Move Bounding Box<br>Resize<br>Quick<br>Invert

## Data preperation

If you have a video file and need the individual frames you can use [ffmpeg](https://ffmpeg.org) to extract the images. This command exports every 10th frame in the video.
`ffmpeg -i input.mp4 -vf "select=not(mod(n\,10))" -vsync vfr 1_every_10/img_%03d.jpg`

## Articles of interest

- [Training YOLOv3 : Deep Learning based Custom Object Detector](https://www.learnopencv.com/training-yolov3-deep-learning-based-custom-object-detector/)

## AWS Preparation

AWS has a free tier for the first 12 months of S3 use (up to 5GB) and DynamoDB is free for up to 25GB. So it should be possible to use the tool for 12 months without cost, if you stick to the restrictions. Further information can be found here [AWS free tier](https://aws.amazon.com/de/free/)

1. Create an [AWS Account](https://portal.aws.amazon.com/billing/signup)
1. Create a DynamoDB Table with the name `ObjectDetectionImageAnnotation`, copy the Amazon Resource Name you find in the overview tab and replace the arn in the following policy. `arn:aws:dynamodb:eu-west-1:XXXXXXX:table/ObjectDetectionImageAnnotation`
1. Create a S3 Bucket and replace the `mys3bucketname` with the new bucketname.
1. Create a new user for this tool in the IAM and add the policy to it below.
1. Change the bucketName, accessKeyId and the secretAccessKey in Alturos.ImageAnnotation.exe.config
1. Start the application

### AWS Policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::mys3bucketname"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::mys3bucketname/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:*"
            ],
            "Resource": [
                "arn:aws:dynamodb:eu-west-1:XXXXXXX:table/ObjectDetectionImageAnnotation"
            ]
        }
    ]
}
```

## An alternative solution so you don't need Amazon AWS

You can also use [MinIO](https://github.com/minio/minio) instead of Amazon S3, alongside a [local DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html)
This way all the files are kept on your PC rather than on a remote server.

### Quick installation

Run `install_local_environment.ps1` using the Windows PowerShell. This should download and set up anything that's necessary.
In case the script doesn't work, you can try a manual setup, as outlined below.

### Manual Installation

* In order to use a local DynamoDB, download it first [here](http://dynamodb-local.s3-website-us-west-2.amazonaws.com/dynamodb_local_latest.zip).
Extract the downloaded zip file into a folder of your choice.

* Next, download and install the Java Runtime Environment [here](https://java.com/download).

* Once installed, set the JAVA_HOME environment variable by opening the command prompt and writing the following:
```
setx JAVA_HOME java_path /M
```
Replace java_path with the path of the "bin" folder in your Java Runtime Environment installation.

* Download and install the AWS CLI from [here](https://aws.amazon.com/de/cli/).

* Write in the command prompt:
```
aws configure
```
You'll be asked to enter some keys. Since the database is entirely local, you can just use fixed demo keys, for instance `AKIAIOSFODNN7EXAMPLE` for the access key id, and `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` for the secret access key.
Remember these keys as they will be used later on again.

* Download [MinIO](https://min.io/download) into a directory of your choice.

* Run the command prompt from the same folder as the downloaded `minio.exe` and write the following:
```
minio.exe server minio
```
Close the command prompt. A folder named `minio` should have been created inside the same directory as your `minio.exe` file.
Navigate to `minio\.minio.sys\config` and open the `config.json` file using a text editor. Change the `accessKey` and `secretKey` on lines 4 and 5 to match the demo keys you chose previously.

The json block should look like this:
```
"credential": {
	"accessKey": "AKIAIOSFODNN7EXAMPLE",
	"secretKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
	"expiration": "1970-01-01T00:00:00Z",
	"status": "enabled"
}
```

### App Config Setup

Once you have installed all the necessary components, you still need to adjust the `Alturos.ImageAnnotation.exe.config` file.

* Change `bucketName` to the bucket name you wish to you use for your database. See the [Amazon S3 Bucket Naming Requirements](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-s3-bucket-naming-requirements.html).

* Change the `accessKeyId` and `secretAccessKey` to the keys AWS is using.
If you did a quick installation, the keys should be `AKIAIOSFODNN7EXAMPLE` and `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` respectively.
If you did a manual installation, use the same keys you chose while configuring AWS earlier.

* Change the `s3ServiceUrl` to `http://localhost:9000`

* Change the `dynamoDbServiceUrl` to `http://localhost:8000`

### The setup is now complete. Launch the run script to use it.

Run `run_local_environment.ps1` using the Windows PowerShell in order to start MinIO and the local DynamoDB.
Once you launch the project you should be able to use your local database.

If it doesn't work, try checking `http://localhost:8000/shell/` in your browser to see if the DynamoDB is running.

## Credits

This program uses icons from the Silk icon set created by Mark James, which can be found [here](http://www.famfamfam.com/lab/icons/silk/).
The icon set is licensed under a [CC BY 3.0 license](https://creativecommons.org/licenses/by/3.0/). Some changes were made to the icons.

## Other Tools

- [rectlabel.com](https://rectlabel.com)
