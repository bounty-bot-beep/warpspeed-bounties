# warpSpeed Bounties

Paid open-source bounty tasks for developers contributing to **warpSpeed OPEN**.

Developers can browse open bounties, sign up on the website, claim a GitHub issue, submit a pull request, and receive payment after the PR is approved and merged.

## Quick Links

- Website: https://warpspeedopen.org
- Bounties: https://warpspeedopen.org/bounties
- Developer Signup: https://warpspeedopen.org/signup
- Discord: Add Discord invite link here
- Community Repo: Add `warpspeed-community` repo link here

## How the bounty process works

1. Browse available bounty tasks.
2. Sign up on the warpSpeed OPEN website.
3. Comment on the GitHub bounty issue to request assignment.
4. Wait for maintainer confirmation before starting paid work.
5. Fork the repository and create your branch.
6. Submit your pull request.
7. Respond to review comments.
8. Get paid after the PR is approved and merged.

## Bounty categories

- Frontend UI
- React Native
- Node.js API
- Prisma / database work
- TypeScript
- AI productivity features
- Documentation
- Integrations

## Important payment rule

Bounties are paid only after the work is approved and the pull request is merged.

## Building the Attachment Summarizer Service

This section outlines the key components and implementation details for the **Attachment Summarizer Service** bounty. The service is built using **Node.js** and **TypeScript**, with integration to **AWS SQS**, **Google Cloud Storage (GCS)**, and a **self-hosted open-source LLM** like **Ollama**.

### 1. Service Architecture Overview

The service is a **Node.js** application that runs as a **Docker container**, consuming messages from **AWS SQS**, downloading attachments from **GCS**, and summarizing content using a **local LLM**. The architecture is designed to be **scalable**, **maintainable**, and **testable**.

### 2. Key Components

#### 2.1. SQS Consumer

The service listens to a **SQS queue** for events that contain **attachment metadata** (e.g., file name, GCS URL, user ID). Here's an example of how to consume messages from SQS using **AWS SDK for JavaScript**:

```typescript
import { SQS } from 'aws-sdk';
import { v4 as uuidv4 } from 'uuid';

const sqs = new SQS({ region: 'us-east-1' });

const consumeMessages = async () => {
  const params = {
    QueueUrl: 'https://sqs.us-east-1.amazonaws.com/123456789012/my-queue',
    MaxNumberOfMessages: 10,
    VisibilityTimeout: 30,
    WaitTimeSeconds: 20,
  };

  const data = await sqs.receiveMessage(params).promise();

  if (data.Messages.length === 0) return;

  for (const message of data.Messages) {
    const { ReceiptHandle, Body } = message;
    const { fileName, gcsUrl, userId } = JSON.parse(Body);

    // Process the attachment
    await processAttachment(fileName, gcsUrl, userId);

    // Delete the message from the queue
    await sqs.deleteMessage({
      QueueUrl: params.QueueUrl,
      ReceiptHandle,
    }).promise();
  }
};
```

#### 2.2. GCS File Download

To download attachments from **GCS**, we use the **Google Cloud SDK**. Here's an example of downloading a file:

```typescript
import { Storage } from '@google-cloud/storage';
import { promises as fs } from 'fs';

const storage = new Storage();

const downloadFile = async (fileName: string, gcsUrl: string) => {
  const file = storage.file(fileName);
  const localPath = `./downloads/${fileName}`;

  await file.download({ destination: localPath });
  console.log(`File ${fileName} downloaded to ${localPath}`);
};
```

#### 2.3. File Content Extraction

The service supports multiple file types, including **PDFs**, **Word documents**, **spreadsheets**, **text files**, **HTML**, and **images**. We use **pdf-lib**, **docxtemplater**, **xlsx**, **htmlparser2**, and **sharp** for this purpose.

#### 2.4. LLM Integration with Ollama

To generate summaries, the service uses a **self-hosted LLM** via **Ollama**. Here's an example of how to send a prompt to the LLM:

```typescript
import { exec } from 'child_process';

const summarizeWithOllama = async (content: string) => {
  const prompt = `Summarize the following text in one concise paragraph:\n\n${content}`;
  const command = `ollama run llama3 "${prompt}"`;

  return new Promise((resolve, reject) => {
    exec(command, (error, stdout, stderr) => {
      if (error) {
        reject(error);
        return;
      }
      resolve(stdout.trim());
    });
  });
};
```

### 3. Docker Setup

The service is containerized using **Docker**. A `Dockerfile` is provided to build the image, and a `docker-compose.yml` file sets up the environment with **AWS CLI**, **Google Cloud CLI**, and **Ollama**.

### 4. Error Handling and Logging

The service includes **error handling** for failed downloads, unsupported file types, and LLM failures. **Logging** is done using **winston** for structured logs.

This service is a robust, scalable solution for summarizing email attachments using open-source technologies.
