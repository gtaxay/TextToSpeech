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
