
# Controling and Integrating the GoPiGo Raspberry Pi robot with AWS IoT using Node.js

The GoPiGo is a complete kit from Dexter Industries to build your own robot car and it has a Raspberry Pi as its brain. When connected to the Internet it can be controlled with the AWS IoT service.

Running this code from the GoPiGo you'll make it an IoT device allowing to control the GoPiGo movements, servo and camera from anywhere as long as the GoPiGo is connected to the Internet via WiFi.

<p align="center">
  <img src="https://raw.githubusercontent.com/DexterInd/GoPiGo/master/GoPiGo_Chassis-300.jpg" />
</p>

# Requirements

* A GoPiGo (http://www.dexterindustries.com/GoPiGo/)
* https://github.com/DexterInd/GoPiGo/tree/master/Software/NodeJS
* https://github.com/aws/aws-iot-device-sdk-js 
* http://docs.aws.amazon.com/iot/latest/developerguide/iot-quickstart.html
* S3 bucket to upload images from the camera (low res due to the 128Kb AWS IoT payload limits)

# Getting Started

* Create a thing called IoTbot using the AWS CLI or the console (https://console.aws.amazon.com/iot/home)
* Create a Policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "iot:*"
      ],
      "Resource": [
        "*"
      ],
      "Effect": "Allow"
    }
  ]
}
```
* Create the certitificates, download the files and attach the certificate to both the policy and the thing created earlier
* Create a S3 rule similar to the one bellow in order to upload files from the camera to the S3 bucket, make sure the IAM role has appropriate access (PutObject). The rule will listen on the 's3upload' topic, upload a JPG file to the bucket and it can be retrieved using a mobile app with access to the bucket (GetObject), for instance:
```
>aws iot get-topic-rule --rule-name S3 

{
    "rule": {
        "sql": "SELECT * FROM 's3upload'",
        "ruleDisabled": false,
        "actions": [
            {
                "s3": {
                    "roleArn": "arn:aws:iam::xxxxxxxxxx:role/aws_iot_s3",
                    "bucketName": "bucket_name",
                    "key": "image.jpg"
                }
            }
        ],
        "ruleName": "S3"
    }
} 
```
* Upload the certificates to your GoPiGo via SSH (http://www.dexterindustries.com/GoPiGo/getting-started-with-your-gopigo-raspberry-pi-robot-kit-2/4-connect-to-the-gopigo/)
* Add your certificates and region details (Line 62) and execute the code:
```
node iotbot.js
```
* Test with a MQTT client like MQTT.fx (http://docs.aws.amazon.com/iot/latest/developerguide/verify-pub-sub.html) by publishing the following to the "$aws/things/IoTbot/shadow/update" topic. Change from "false" to "true" to test the commands accordingly:

```
{
"state": {
      "reported": {
        "stop": "true",
        "forward": "false",
        "left": "false",
        "right": "false",
        "back": "false",
        "picture": "false",
        "lookLeft": "true",
        "lookRight": "false"
      }
    }
}
```
* I am also working on a sample Android app that publishes to the same topic as well as retrieves the image from the S3 bucket. I'll make it available in a separate repository when ready.
