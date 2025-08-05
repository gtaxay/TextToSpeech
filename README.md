# PollyTranslate

**PollyTranslate** is an AWS Lambda function that takes in a text string, converts it to speech using Amazon Polly, and uploads the resulting `.mp3` audio file to an S3 bucket.

---

## 📌 Overview

This project is built around serverless architecture using AWS services. It leverages **Amazon Polly** to synthesize speech from plain text, and stores the audio file in an **S3 bucket** for easy access. The audio is streamed directly to S3, so there’s no need to store anything locally.

---

## How It Works

1. The Lambda function receives an event containing a `text` field.
2. Amazon Polly converts the text into an `.mp3` audio stream.
3. The audio stream is uploaded to a specified S3 bucket (`polly-translate-storage`) using the AWS SDK's streaming utility.
4. A success response is returned with the filename of the uploaded audio.

---

## Code

```js
const { PollyClient, SynthesizeSpeechCommand } = require("@aws-sdk/client-polly");
const { S3Client } = require("@aws-sdk/client-s3");
const { Upload } = require("@aws-sdk/lib-storage"); 

const polly = new PollyClient({});
const s3 = new S3Client({});

exports.handler = async (event) => {
    try {
        const text = event.text;

        const params = {
            Text: text,
            OutputFormat: "mp3",
            VoiceId: "Joanna",
        };

        const command = new SynthesizeSpeechCommand(params);
        const data = await polly.send(command);

        const key = `audio-${Date.now()}.mp3`;

        const upload = new Upload({
            client: s3,
            params: {
                Bucket: "polly-translate-storage", 
                Key: key,
                Body: data.AudioStream, 
                ContentType: "audio/mpeg",
            },
        });

        await upload.done();

        return {
            statusCode: 200,
            body: JSON.stringify({ message: `Audio file stored as ${key}` }),
        };
    } catch (error) {
        console.error("Error:", error);
        return {
            statusCode: 500,
            body: JSON.stringify({ message: "Internal server error" }),
        };
    }
};

## Testing

{
  "text": "Hello from AWS Polly!"
}

## Requirements
- AWS Lambda (Node.js 18.x or later)
- An S3 bucket (e.g., polly-translate-storage)
- IAM role attached to the Lambda function with the following permissions:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "polly:SynthesizeSpeech",
        "s3:PutObject"
      ],
      "Resource": "*"
    }
  ]
}

## Libraries Used
- @aws-sdk/client-polly
- @aws-sdk/client-s3
- @aws-sdk/lib-storage
