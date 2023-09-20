# Flask API for cropping images and uploading them to storage buckets

create a `Dockerfile` to containerize the Flask App

```Dockerfile
# Use an official Python runtime based on Debian 10 as a parent image
FROM python:3.10-slim-buster

# Set the working directory in the container to /app
WORKDIR /app

# Add the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 8080 available to the world outside this container
EXPOSE 8080

# Define path to keyfile as an environment variable
ENV KEYFILE_PATH service-account-keyfile.json

# Run app.py when the container launches
CMD exec gunicorn --bind :8080 --workers 1 --threads 8 --timeout 0 main:app
```

The `requirement.txt` file would look something like this

```js
Flask==2.3.3
gunicorn==21.2.0
opencv-python==4.8.0.76
```

Here's a simple `cloudbuild.yaml` file for deploying a Flask application to Google Cloud Run using Google Cloud Build:

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-flask-app', '.']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/my-flask-app']
- name: 'gcr.io/cloud-run/button'
  args: ['deploy', 'my-flask-app', '--image', 'gcr.io/$PROJECT_ID/my-flask-app', '--region', 'us-central1', '--platform', 'managed']
```

In this file:

- The `build` step builds the Docker image from the Dockerfile in the current directory.
- The `push` step pushes the Docker image to Google Container Registry.
- The `deploy` step deploys the Docker image to Google Cloud Run.

Replace `my-flask-app` with the name of your Flask application, and replace `us-central1` with the region where you want to deploy your application.

To set up a Continuous Deployment (CD) pipeline with GitHub and Google Cloud Build, follow these steps:

1. Go to the Google Cloud Console.

2. Navigate to Cloud Build > Triggers.

3. Click "Connect Repository", and follow the instructions to connect to your GitHub repository.

4. After connecting your repository, click "Create Trigger".

5. In the "Create Trigger" form, enter a name for the trigger, and choose the event that should trigger the build (for example, a push to the main branch).

6. In the "Build Configuration" section, select "Cloud Build configuration file (yaml or json)", and enter the location of your `cloudbuild.yaml` file in your repository.

7. Click "Create" to create the trigger.

Now, every time the specified event occurs in your GitHub repository, Google Cloud Build will automatically build and deploy your Flask application to Google Cloud Run.

Alternatively, you can setup the CD pipeline directly using Github Actions

To set up a GitHub Actions workflow for Continuous Deployment (CD) to Google Cloud Run, you can follow these steps:

1. **Create a Service Account in Google Cloud Platform (GCP):**

   - Go to the Google Cloud Console.
   - Navigate to `IAM & Admin` > `Service Accounts`.
   - Click `Create Service Account`, fill in the details and create the account.
   - In the `Service account permissions` section, add the `Cloud Run Admin` role (for deploying to Cloud Run) and the `Service Account User` role (for letting Cloud Build impersonate the service account).
   - Create a key for this service account (in JSON format) and download it. This key file will be used to authenticate GitHub Actions with GCP.

2. **Add the Service Account Key as a Secret in GitHub:**

   - Go to your GitHub repository.
   - Navigate to `Settings` > `Secrets`.
   - Click `New repository secret`.
   - Name the secret (e.g., `GCP_SA_KEY`) and paste the entire content of the service account key file into the value field. Click `Add secret`.

3. **Create a GitHub Actions Workflow:**

   - In your GitHub repository, create a new file in the `.github/workflows` directory (create the directories if they don't exist). You can name the file as you like, but it must end with `.yml` (e.g., `deploy.yml`).
   - In this file, define your GitHub Actions workflow. Here's a basic example:

```yaml
name: Deploy to Google Cloud Run

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@master
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
        export_default_credentials: true

    - name: Deploy to Cloud Run
      run: |
        gcloud run deploy my-service --image gcr.io/my-project/my-image --region us-central1 --platform managed --allow-unauthenticated
```

In this workflow:

- The `on` section specifies that the workflow should be triggered on a push to the `main` branch.
- The `checkout` step checks out your repository.
- The `Set up Cloud SDK` step sets up the Google Cloud SDK, authenticates with the service account key, and sets the project ID.
- The `Deploy to Cloud Run` step deploys your application to Cloud Run.

Make sure to replace `my-service` with the name of your Cloud Run service, `gcr.io/my-project/my-image` with the path to your Docker image in Google Container Registry, and `us-central1` with the region where you want to deploy your service.

Also, add your GCP project ID as a secret (e.g., `GCP_PROJECT_ID`) in GitHub, similar to how you added the service account key.

4. **Commit and Push the Workflow:**

   - Commit the workflow file and push it to your GitHub repository. The workflow will be triggered on the next push