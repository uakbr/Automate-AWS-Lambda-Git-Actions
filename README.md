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

##Create GitHub Actions Workflow Directory
* To create the CI/CD workflow in GitHub Actions, we need to create a `.yml file` in our repository. From our application root, create a folder named `.github/workflows` that will contain the GitHub action workflows.
* Then, create `main.yml` (just an example, we can put any name that you like for the .yml file) inside the created folder as this file will contain all the instructions for our automated tests and deployment process to AWS Lambda through our code on GitHub repository. You can use the code below in the terminal as the guide to achieve this process.

```
cd path/to/root_repo
mkdir .github/workflows
touch .github/workflows/main.yml
```
Cool! Now we have the directory for GitHub Actions workflow.
