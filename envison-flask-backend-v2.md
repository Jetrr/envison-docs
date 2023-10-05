# API Documentation

This documentation provides information on how to use the API for the client-side developer.

## Initialize Job

**Endpoint:** `/initalize-job`

**Method:** `GET`

**Description:** This endpoint is used to initialize a new job. 

**Example Request:**

```http
GET /initalize-job
```

**Example Response:**

```json
{
    "jobId": "1234"
}
```

## Upload Image

**Endpoint:** `/upload`

**Method:** `POST`

**Description:** This endpoint is used to upload an image for a specific job.

**Request Body:**

- `jobId`: The ID of the job.
- `image`: The image file to be uploaded.

**Example Request:**

```http
POST /upload
Content-Type: multipart/form-data;
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg
```

**Example Response:**

```json
{
    "message": "Image uploaded successfully",
    "numImages": 1,
    "jobId": "1234"
}
```

## Start Training

**Endpoint:** `/start-training/<jobId>`

**Method:** `POST`

**Description:** This endpoint is used to start the training process for a specific job.

**Example Request:**

```http
POST /start-training/1234
```

**Example Response:**

```json
{
    "jobId": "1234"
}
```

## Get Job Status

**Endpoint:** `/job/<jobId>`

**Method:** `GET`

**Description:** This endpoint is used to get the status of a specific job.

**Example Request:**

```http
GET /job/1234
```

**Example Response:**

```json
{
    "status": "Completed",
    "urls": [
        "https://example.com/image1.jpg",
        "https://example.com/image2.jpg"
    ]
}
```

## Get All Jobs

**Endpoint:** `/all-jobs`

**Method:** `GET`

**Description:** This endpoint is used to get the list of all jobs.

**Example Request:**

```http
GET /all-jobs
```

**Example Response:**

```json
{
    "jobs": [
        {
            "jobId": "1234",
            "status": "Completed",
            "numImages": 2
        },
        {
            "jobId": "5678",
            "status": "Pending",
            "numImages": 1
        }
    ]
}
```

## Client Side Workflow

a pseudo code representing the general workflow of the API calls:

```python
# Initialize a new job
response = requests.get('https://api.example.com/initalize-job')
jobId = response.json()['jobId']

# Upload images
for i in range(10):
    image_file = open(f'image{i}.jpg', 'rb')
    files = {'image': image_file}
    data = {'jobId': jobId}
    response = requests.post('https://api.example.com/upload', files=files, data=data)
    if response.status_code != 200:
        print(f"Error uploading image{i}.jpg: {response.json()['message']}")
    else:
        print(f"Successfully uploaded image{i}.jpg")

# Start training
response = requests.post(f'https://api.example.com/start-training/{jobId}')
if response.status_code != 200:
    print(f"Error starting training: {response.json()['message']}")
else:
    print(f"Training started for job {jobId}")

# Check job status
response = requests.get(f'https://api.example.com/job/{jobId}')
if response.status_code != 200:
    print(f"Error getting job status: {response.json()['message']}")
else:
    status = response.json()['status']
    print(f"Job status: {status}")
    if status.lower() == 'completed':
        print(f"Image URLs: {response.json()['urls']}")
```

## Impementing a Swift Client

Here is a basic implementation in SwiftUI. This implementation assumes that you have a `Job` model and a `NetworkManager` class to handle API calls.

```swift
import SwiftUI

struct ContentView: View {
    @State private var jobId: String = ""
    @State private var images: [UIImage] = []
    @State private var jobStatus: String = ""
    @State private var imageURLs: [String] = []

    var body: some View {
        VStack {
            Text("Job ID: \(jobId)")
            Text("Job Status: \(jobStatus)")
            List(imageURLs, id: \.self) { url in
                Text(url)
            }
            Button("Initialize Job") {
                NetworkManager.shared.initializeJob { (jobId) in
                    self.jobId = jobId
                }
            }
            Button("Upload Image") {
                // Assuming you have a method to get images
                let image = getAnImage()
                NetworkManager.shared.uploadImage(jobId: self.jobId, image: image) { (success) in
                    if success {
                        self.images.append(image)
                    }
                }
            }
            Button("Start Training") {
                NetworkManager.shared.startTraining(jobId: self.jobId)
            }
            Button("Get Job Status") {
                NetworkManager.shared.getJobStatus(jobId: self.jobId) { (status, urls) in
                    self.jobStatus = status
                    self.imageURLs = urls
                }
            }
        }
    }
}
```

And here is a basic implementation of the `NetworkManager` class:

```swift
import Foundation

class NetworkManager {
    static let shared = NetworkManager()
    private init() {}

    func initializeJob(completion: @escaping (String) -> Void) {
        // Call the /initialize-job API and pass the jobId to the completion handler
    }

    func uploadImage(jobId: String, image: UIImage, completion: @escaping (Bool) -> Void) {
        // Call the /upload API with the jobId and image, and pass the success status to the completion handler
    }

    func startTraining(jobId: String) {
        // Call the /start-training/{jobId} API
    }

    func getJobStatus(jobId: String, completion: @escaping (String, [String]) -> Void) {
        // Call the /job/{jobId} API and pass the status and URLs to the completion handler
    }
}
```