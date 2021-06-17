## AWS Lambda FastAPI CI/CD Pipeline using GitHub Actions
This project repository contains the github actions workflow of CI/CD Pipeline for FastAPI to AWS Lambda.

## Steps:
1. Create a GitHub Repository
2. Clone your repository to local machine
3. Setup Virtual Environment
4. Install the Required Dependencies
5. Run the API in Local Machine
6. Run Unit Test in Local Machine
7. Update requirements.txt
8. Create GitHub Actions Workflow Directory
9. The Components of GitHub Actions
10. Continuous Integration (CI): Build Automated Test
11. Configuring GitHub Secrets, Amazon S3 and AWS Lambda
12. Continuous Deployment (CD): Deploy Lambda
13. Running the GitHub workflow

## Create GitHub Actions Workflow Directory
* To create the CI/CD workflow in GitHub Actions, we need to create a `.yml file` in our repository. From our application root, create a folder named `.github/workflows` that will contain the GitHub action workflows.
* Then, create `main.yml` (just an example, we can put any name that you like for the .yml file) inside the created folder as this file will contain all the instructions for our automated tests and deployment process to AWS Lambda through our code on GitHub repository. You can use the code below in the terminal as the guide to achieve this process.

```
cd path/to/root_repo
mkdir .github/workflows
touch .github/workflows/main.yml
```
Cool! Now we have the directory for GitHub Actions workflow.

## Understanding the GitHub Actions Workflow
There are six main components in GitHub Actions:
* **Workflow** — The automated procedure that we add to our repository and can be triggered or scheduled by an event.
* **Events** — It is a specific activity that will trigger a workflow. For example, an event triggers when someone pushes a commit to a repository or a pull request is created.
* **Jobs** —Series of steps that execute on the same runner. It can run parallelly or sequentially depends on our workflow objectives.
* **Steps** — It is an individual task that will run commands in a job.
* **Actions** — An action is a set of standalone commands that gets executed on the runner and it is combined into steps to create a job.
* **Runners** — A server that hosted virtual operating systems by GitHub or your own (self-hosted) that can run commands to perform a build process. Hosted runners by GitHub are based on Microsoft Windows, Ubuntu Linux and macOS.

The workflow that we are going to build will consist of two main jobs:
* **Continuous Integration (CI)**

The CI will run the automated test, package our FastAPI into Lambda and upload the lambda artifact in the GitHub server in order to enable the other jobs (in our case, the Continuous Deployment Job) to use it.

**Continuous Deployment (CD)**
This job will only be executed when the CI job is successfully completed. CD job needs to be dependent on the status of the CI build job in order to make sure that we can only deploy the application once we have passed the CI part. Basically, this job will download the lambda artifact that has been uploaded during the CI job and deploy it to AWS Lambda by linking it with AWS S3.

So our workflow will look like this in the `.yml` file:
```
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
jobs:

  continuous-integration:
  ....
  
  continuous-deployment:
  .....
```

For the detailing part of the steps and commands inside each job, please stay along with me okay!
Alright, let’s get our hands dirty by creating the workflow using GitHub Actions.

## Continuous Integration (CI): Build Automated Test and Package Lambda
Here is the complete CI workflow in our `main.yml` file.
```
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]


jobs:

  continuous-integration:
    runs-on: ubuntu-latest

    steps:
      # Step 1      
      - uses: actions/checkout@v2
      
      # Step 2
      - name: Set up Python 
        uses: actions/setup-python@v2
        with:
          python-version: 3.7
          architecture: x64
      # Step 3
      - name: Install Python Virtual ENV
        run: pip3 install virtualenv
      # Step 4
      - name:  Setup Virtual env
        uses: actions/cache@v2
        id: cache-venv
        with:
          path: venv
          key: ${{ runner.os }}-venv-${{ hashFiles('**/requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-venv-
      # Step 5
      - name: Activate and Install Depencies into Virtual env
        run: python -m venv venv && source venv/bin/activate &&
          pip3 install -r requirements.txt
        if: steps.cache-venv.outputs.cache-hit != 'true'
      # Step 6     
      - name: Activate venv and Run Test        
        run: . venv/bin/activate && pytest
      
      # Step 7
      - name: Create Zipfile archive of Dependencies
        run: |
          cd ./venv/lib/python3.7/site-packages
          zip -r9 ../../../../api.zip .
      
      # Step 8
      - name: Add App to Zip file
        run: cd ./app && zip -g ../api.zip -r .
      
      # Step 9
      - name: Upload zip file artifact
        uses: actions/upload-artifact@v2
        with:
          name: api
          path: api.zip
```

Let us break it down and have a look at each part of the workflow:
* The name assigned to this workflow is `CI/CD Pipeline`
* The workflow will be triggered when commit codes pushed to the main branch in the repository.
* The job defined in this workflow is continuous-integration.
* The runner used in the workflow is ubuntu-latest (Ubuntu Linux Operating Systems)
* These are the sequential series of steps defined in the CI workflow:

**Step 1:** Perform `actions/checkout@v2` that will checkout to our repository and downloads it to the runner.
**Step 2:** Setup python 3.7 by using actions - `actions/setup-python@v2`
**Step 3:** Install Python Virtual Environment (virtualenv) package
**Step 4:** Caching Dependencies using `actions/cache@v2.` This step will increase the performance of the project workflow that consists of many large dependencies by efficiently reduce the time required for downloading. For further explanation, please read the GitHub Actions Documentations.
**Step 5:** Activate the virtualenv and install all the dependencies that consist inside `requirements.txt`.
**Step 6:** Run Unit Tests. By the way, we need to activate again the virtualenv before running the test as GitHub Actions doesn’t preserve the environment.
**Step 7:** Package our Lambda by zipping all the dependencies in the venv site-packages and place it in our root directory. The zip file is named `api.zip`.
**Step 8:** Add the contents of our app folder into `api.zip`.
**Step 9:** Upload the api.zip to GitHub server as an artifact using `actions/upload-artifact@v2`. This will enable the next job to retrieve back the artifact file for the deployment of our Lambda package which is our `api.zip` file.

Phew! We have completed our CI workflow using GitHub Actions. Let’s make some minor changes in our code and try to push our code to the main branch. Make sure you’re using the CI code above in the `main.yml` file to test the CI workflow. This is how the result will look like when we have trigger the workflow by our code push event.
