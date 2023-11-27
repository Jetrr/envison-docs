# BenchMarking Creating Signed URLs with different languages

An important thing to note here is that the code should not use any kind of IO like printing to the terminal between the `time` end points that are benchmarked.

## Results

| Language | Time (seconds) | URLs per second | Slower than Go |
|----------|-----------------|-----------------|---------------|
| Python   | 0.02s           | 50              |25x            |
| Node     | 0.0013s         | 770             |1.6x           |
| Go       | 0.0008s         | 1250            |1x             |

- `Python` is `25` times slower than `Go` and `16` times slower than `Node Js`.
- To visualize how the performance difference affects user experience, if we suppose that an endpoint is supposed to generate `200` urls in a single requests, python would take `4+` seconds while NodeJs can complete the request within a `500ms`
- Note that these benchmark were done on a local environment, in cloud environment where CPU is virtualized these times can increase by a reasonable magnitude. 

## Code Snippets

### 1. Node

```ts
import { Storage } from "@google-cloud/storage";
import env from "./env";

const storage = new Storage({
    keyFilename: "keyfile.json",
})

async function main() {
    const bucket = storage.bucket(env.BUCKET_NAME + "-dev")
    async function getPresignedUrl(blobName: string) {
        const blob = bucket.file(blobName)
        const [url] = await blob.getSignedUrl({
            action: "read",
            expires: Date.now() + 1000 * 60 * 60 * 24 * 7,
        })
        return url
    }
    const startTime = Date.now()
    const ITERS = 10000;
    for (let i = 0; i < ITERS; i++) {
        const url = await getPresignedUrl(`test${i}.txt`)
    }
    const totalTime = Date.now() - startTime

    console.log(`Average time: ${totalTime / ITERS / 1000}s`)
}

main()
```

### 2. Python

```py
from google.cloud import storage
from constants import *
import os
from datetime import timedelta, datetime

os.environ.setdefault("GOOGLE_APPLICATION_CREDENTIALS", "keyfile.json")

client = storage.Client()

bucket = client.get_bucket(BUCKET_NAME)

def get_signed_url(jobId: str, image_num: int) -> str:
    blob = bucket.blob(f"{jobId}/output/subject-{image_num}.png")
    url = blob.generate_signed_url(
        version="v4",
        expiration=timedelta(minutes=15),
        method="GET",
    )
    return url

# benchmarking
ITERS = 10000

startTime = datetime.now()
for i in range(ITERS):
    get_signed_url("test", i)

print(f"Average time to get signed url: {(datetime.now() - startTime) / ITERS}")
```

### 3. Golang

```go
package main

import (
	"context"
	"fmt"
	"time"
	"cloud.google.com/go/storage"
	"google.golang.org/api/option"
)

const (
	bucketName = "envison-us-central1-dev" // Replace with your bucket name
)

func getSignedURL(ctx context.Context, client *storage.Client, jobId string, imageNum int) (string, error) {
	bkt := client.Bucket(bucketName)
	obj := bkt.Object(fmt.Sprintf("%s/output/subject-%d.png", jobId, imageNum))

	opts := &storage.SignedURLOptions{
		GoogleAccessID: "559809127041-compute@developer.gserviceaccount.com", 
		PrivateKey:     []byte("-----BEGIN PRIVATE KEY-----\n(keyhere)-----END PRIVATE KEY-----\n"),
		Method:         "GET",
		Expires:        time.Now().Add(15 * time.Minute),
	}

	url, err := storage.SignedURL(bucketName, obj.ObjectName(), opts)
	if err != nil {
		return "", err
	}

	return url, nil
}

func main() {
	ctx := context.Background()
	client, err := storage.NewClient(ctx, option.WithCredentialsFile("keyfile.json"))
	if err != nil {
		panic(err)
	}
	defer client.Close()

	// Benchmarking
	const iters = 10000
	startTime := time.Now()

	for i := 0; i < iters; i++ {
		_, err := getSignedURL(ctx, client, "test", i)
		if err != nil {
			fmt.Println("Error getting signed url: ", err)
			panic(err)
		}
	}

	avgTime := time.Since(startTime) / iters
	fmt.Printf("Average time to get signed url: %v\n", avgTime)
}
```