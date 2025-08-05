# PollyTranslate

**PollyTranslate** is an AWS Lambda function that takes in a text string, converts it to speech using Amazon Polly, and uploads the resulting `.mp3` audio file to an S3 bucket.

---

## 📌 Overview

This project is built around serverless architecture using AWS services. It leverages **Amazon Polly** to synthesize speech from plain text, and stores the audio file in an **S3 bucket** for easy access. The audio is streamed directly to S3, so there’s no need to store anything locally.

---

## Services Used

- **Amazon Polly** – Turns text into realistic speech
- **AWS S3** – Stores the audio files
- **AWS Lambda** – Runs the code serverlessly
- **AWS IAM** – Handles secure permissions

---

## How It Works

1. You send a request to the Lambda function with a `text` field.
2. Polly takes the text and generates an `.mp3` audio stream.
3. The audio stream is uploaded directly to your S3 bucket (`polly-translate-storage`).
4. Lambda returns the filename of the audio file so you can grab it from S3.

---

## Code

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

---

## Testing

    {
      "text": "Hello from AWS Polly!"
    }

---

## Requirements

- AWS Lambda (Node.js 18.x or later)
- An S3 bucket (e.g., `polly-translate-storage`)
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

---

## Libraries Used
- @aws-sdk/client-polly
- @aws-sdk/client-s3
- @aws-sdk/lib-storage

---

## Example Input

    {
	"text":"Hi, I’m Gabe, just a guy who gave AWS a voice. You're listening to the result. "
    }
