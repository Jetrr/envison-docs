## Our Inital Approach

### On Backend

Backend remotely starts a virtual machine with gpu that executes training code.

```javascript
// API hosted on https://our-backend-endpoint.dev/start-fine-tune
function finetuneRequestHandler() {
    const subjectImage = userRequest.subjectImageLink

    // create a VM with GPU allocated to run training code
    const node = computeEngineAPI.createMachine({
        acceleratorType: "gpu",
        customContainer: "/link/to/custom-training-container-image"
        parameters: {
            subjectImage: "/gcs/bucket-name/image-folder"
        }
    })

    //When job is done, free up the gpu node
    node.delete()

    // Call to database happens here for keeping track of the ml job status
    Database.update({ status: "completed" })
}
```

### Client Side
The client-side is supposed to call the fine tune api after it has uploaded

It can be a web app or a mobile app:
#### Clients
- **Web App:** Javascript
- **IOS App:** Swift
- **Android App:** Java

#### IOS APP (Swift)
```swift
// Upload Images to Bucket
var bucketLink = UploadImageToBucket(images)
// Submit a backend request to a finetuning api
SubmitFineTuningRequest(
    url: "https://our-backend-endpoint.dev/start-fine-tune", 
    imagesLink: bucketLink
)
```
or
#### Web APP (Javascript)
```javascript
// Upload Images to Bucket
const bucketLink = UploadImageToBucket(images)
// Submit a backend request to a finetuning api
axios.post(
    "https://our-backend-endpoint.dev/start-fine-tune",
    { imagesLink: bucketLink }    
)
```

At the end of the day, it does not matter what client-side language we are using, all that is required from the client side is to upload the images to somewhere and makes an HTTP call to the BFF (Backend For Frontend). The BFF is a cloud function which may be implemented in Javascript or Python.

This model allows us to provide a simple interface for the client-side application developer and abstract functionality. This also provides modularity as we are implementing the backend logic separately from the client-side application.

By separating the frontend (client-side) and backend (BFF) code, we can develop and maintain them independently. This allows for easier maintenance and scalability as changes in one component do not affect the other.

## Our Current Approach With Vertex AI

We develop a ml pipeline using `kubeflow` in python and deploy it on vertex ai

It looks something like this
```python
from kfp.dsl import compiler

def pipeline():

    def trainingJob():
        gcloud.ai.createJob({
            "jobID": 123,
            "gpu": "T4",
            "customContainer": "/link/to/custom-training-container-image"
        })
        updateFireStoreDatabase({ "status": "completed" })

# using kubeflow dsl (domain-specific language), We create a Template in YAML format
yamlFile = compiler.Compile(pipeline)

# Then we take the yaml template file and deploy it to Vertex AI as a Pipeline
gcloud.ai.deployPipeline(yamlFile)
```

### BFF (Backend For Frontend) Code

We would recieve requests from our client-side application
On Fine tune requests from client, we will trigger our pipeline on Vertex AI using it's API 

```javascript
// API hosted on https://our-backend-endpoint.dev/start-fine-tune
function finetuneRequestHandler() {
    const subjectImage = userRequest.subjectImageLink

    // Trigger the ML Pipeline which would run on Vertex AI
    const node = vertexAiAPI.runPipeline({
        pipelineID: "dreambooth-training-ml-pipeline",
        parameters: {
            subjectImage: "/gcs/bucket-name/image-folder"
        }
    })

    // notify the client once the images are generated
    notifyClient()
}
```

In this case, we are not dealing with database records or things like hyperparameter finetuning in the BFF because the training pipeline on vertexAI takes care of that.

### Client Side Implementation

The client side implementation should be the same as in our initial approach. The basic idea is that even if we make changes/development to our backend logic, client side code should not have to be modified. The exposed interface by BFF APIs for the client side should stay the same. Only the underlying logic/code would change.