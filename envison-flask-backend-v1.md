# API Documentation

## Upload API

### Request

- **Endpoint:** `/upload`
- **Method:** `POST`
- **Body:** A `multipart/form-data` where `images` field contains the images to be uploaded.

### Response

- **Success (200):** Returns a JSON object containing the `jobId` for the upload job.
- **Failure (400):** Returns a JSON object with a `message` detailing the error.

```json
{
    "jobId": "1234"
}
```

## Job Status API

### Request

- **Endpoint:** `/job/<jobId>`
- **Method:** `GET`
- **Parameters:** `jobId` (Path variable)

### Response

- **Success (200):** Returns a JSON object containing the `status` of the job. If the job is completed, it also includes an array of `urls` for the processed images.
- **Failure (404):** Returns a JSON object with a `message` detailing the error.

```json
{
    "status": "completed",
    "urls": ["https://example.com/image1", "https://example.com/image2"]
}
```

## Get All Jobs

**Endpoint:** `/all-jobs`

**Method:** `GET`

**Description:** This endpoint is used to get a list of all jobs.

**Responses:**

- `200 OK` with a list of all jobs.

## Usage

1. Upload the images through the `/upload` API. If the response contains a `jobId`, store it locally on the client side.
2. Every 2 minutes, call the `/job/<jobId>` API to check the status of the job.
3. If the status is `completed`, you will receive an array of pre-signed URLs for the processed images.
