---
layout: post
title: "Optimizing File Upload Architecture with AWS S3 Presigned POST"
excerpt: Reduce infrastructure strain by cutting the backend out of the upload path.
date: 2025-07-12T16:00:10+02:00
comments: true
tags: [aws, s3, performance, architecture, presigned-url, optimization]
image:
  feature: posts/presigned-post/cover-upload.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

In many systems, file uploads follow a familiar pattern:

1. A client sends the file to your application server.
2. The backend processes it (e.g., for validation or virus scanning).
3. Then it forwards the file to a cloud storage service like Amazon S3.

While this works, it’s highly inefficient. Your app becomes a middleman consuming bandwidth, CPU, and memory, just to pass files along.

## Why This Is a Problem

This upload pattern:

- **Consumes resources** on your app server.
- **Increases latency** for users.
- **Stresses your infrastructure** (especially under load).
- **Breaks down for large files**, especially when you hit limits in intermediaries like API Gateway.

Let’s look at two common architectures where these problems show up.

## Two Upload Architectures Compared

### 🔸 Architecture 1: Upload with Nginx

This is typical in smaller-scale deployments with a single server behind a reverse proxy.

<div class="mermaid">
flowchart LR
  Client["Client (Browser/App)"]
  Nginx["Nginx (Reverse Proxy)"]
  App["App Server"]
  S3["S3 Bucket"]

  Client --> Nginx
  Nginx --> App
  App --> S3
</div>

**Problems:**

- Nginx must **buffer** the file in memory or disk (an exception can be thrown here).
- App still receives and processes the full payload.
- Memory and CPU are wasted just to forward the upload.

### 🔸 Architecture 2: Upload via API Gateway + Load Balancer

This setup is common in cloud-native apps using AWS infrastructure.

<div class="mermaid">
flowchart LR
  Client["Client (Browser/App)"]
  APIGW["AWS API Gateway"]
  ALB["Application Load Balancer"]
  App["App Server"]
  S3["S3 Bucket"]

  Client --> APIGW
  APIGW --> ALB
  ALB --> App
  App --> S3
</div>

**Problems:**

- **API Gateway limits uploads to 10 MB** max per request.
- Both ALB and your app must handle the file payload.
- This creates bottlenecks and risks timeouts.

## ✅ The Solution: Presigned POST to S3

Instead of routing uploads through your backend, you can **let clients upload directly to S3**, securely, using a **presigned POST policy**.

### How It Works

1. The client asks your backend for a signed upload policy.
2. Your backend returns a temporary URL and form fields.
3. The client uploads the file directly to S3 using `multipart/form-data`.

<div class="mermaid">
sequenceDiagram
  autonumber
  participant Client as Client (Browser/App)
  participant API as App Server
  participant S3 as S3 Bucket

  Client->>API: GET /attachments/presigned-post-policy
  API-->>Client: 200 OK (URL + fields)
  Client->>S3: POST file upload (multipart/form-data)
  S3-->>Client: 204 No Content
</div>

## Benefits of Presigned Uploads

- 🚀 **Bypass Infrastructure**: No more file handling by Nginx, ALB, or API Gateway.
- 📉 **Reduce Server Load**: No memory or CPU usage from streaming file data.
- ⏱️ **Improve Latency**: Fewer hops and no buffering.
- 💡 **Avoid Limits**: Works around API Gateway’s 10MB payload cap.
- 🔐 **Secure**: You control what, where, and for how long uploads are allowed.

## Example Server-Side implementation

```typescript
import { Injectable } from '@nestjs/common';
import { S3Client } from '@aws-sdk/client-s3';
import { createPresignedPost } from '@aws-sdk/s3-presigned-post';

@Injectable()
export class S3Service {

  public getPresignedPostPolicy(
    filename: string,
    bucketConfig: S3BucketResourceConfig,
    contentType?: string
  ): Promise<{ url: string; fields: Record<string, string> }> {
    const expiresInSeconds = 600; // Default 10 minutes
    const maxSizeBytes = 150 * 1024 * 1024; // Default 150 MB

    const s3ClientConfig: S3ClientConfig = {
      region: bucketConfig.region
    };

    return createPresignedPost(new S3Client(s3ClientConfig), {
      Bucket: bucketConfig.name,
      Key: filename,
      Conditions: [
        ['content-length-range', 0, maxSizeBytes],
        ['starts-with', '$Content-Type', contentType || '']
      ],
      Fields: {
        ...(contentType && { 'Content-Type': contentType })
      },
      Expires: expiresInSeconds
    });
  }

}
```

## Example Client Implementation

Here’s how a client might perform the upload:

```typescript
import axios from 'axios';

async function uploadFile(file: File) {
  // Step 1: Get the presigned policy
  const res = await axios.get('/api/get-presigned-policy', {
    params: {
      filename: file.name,
      contentType: file.type,
    },
  });

  const { url, fields } = res.data;

  // Step 2: Prepare the upload form
  const formData = new FormData();
  Object.entries(fields).forEach(([key, val]) => formData.append(key, val as string));
  formData.append('file', file); // must be last

  // Step 3: Upload to S3
  const uploadRes = await axios.post(url, formData, {
    headers: { 'Content-Type': 'multipart/form-data' },
  });

  if (uploadRes.status === 204) {
    console.log('Upload successful');
  } else {
    console.error('Upload failed');
  }
}
```

## When Should You Use Presigned POST?

- When handling **large file uploads**.
- When deploying behind **API Gateway**.
- When you want to **scale uploads** without scaling your backend.
- When you want **better performance** and **lower infrastructure costs**.

## Summary

If you're still uploading files through your backend, you're likely wasting resources, risking timeouts, and introducing unnecessary complexity.

Presigned POST policies in AWS S3 offer a better approach:

- Simple to implement.
- Secure and time-limited.
- Scalable and resilient.

Offload uploads to your storage layer, which is where they belong, and keep your app fast, simple, and maintainable.

## More info about the AWS feature

- [Browser-Based Uploads Using POST (AWS Signature Version 4](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-UsingHTTPPOST.html)
- [POST Policy - Amazon Simple Storage Service](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-HTTPPOSTConstructPolicy.html)
- [Example: Browser-Based Upload using HTTP POST (Using AWS Signature Version 4)](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-post-example.html)
