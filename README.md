# GitOps-> Gitlab_Kubernetes

In our previous blog post, we delved into the process of constructing a Docker image utilizing GitLab for seamless CI/CD integration. Building upon that foundation, let's now explore the next steps: deploying these Docker images into the container registry within GitLab and establishing connectivity with a Kubernetes cluster through the utilization of GitLab agents.

Deploying Docker images into GitLab's container registry provides a centralized repository for our containerized applications. This registry serves as a secure and efficient storage solution, ensuring easy access and version control for our Docker images.

Furthermore, by leveraging GitLab agents, we can seamlessly connect our Kubernetes cluster with the GitLab environment. These agents act as intermediaries, facilitating communication and orchestration between GitLab's CI/CD pipelines and the Kubernetes infrastructure. This integration streamlines the deployment process, enabling automated scaling, monitoring, and management of our containerized applications within the Kubernetes environment.

Stay tuned as we delve deeper into the intricacies of deploying Docker images to GitLab's container registry and establishing seamless connectivity with Kubernetes through GitLab agents, unlocking the full potential of modern DevOps practices.

Here’s a breakdown of GitLab, its CI/CD functionality, and key concepts like jobs, scripts, and stages:

# GitLab

GitLab is a complete DevOps platform delivered as a single application. It provides a Git repository manager, wiki, issue tracking, CI/CD pipelines, and more. GitLab enables teams to collaborate on code, manage projects, and automate software delivery pipelines.

# CI/CD in GitLab:

# Continuous Integration (CI):
CI is a software development practice where developers regularly merge their code changes into a central repository, after which automated builds and tests are run. GitLab CI allows developers to define pipelines as code, automating the build, test, and deployment processes for their applications.

# Continuous Deployment (CD):
CD is the practice of automatically deploying code changes to production environments after passing through the CI process. GitLab provides CD capabilities alongside CI, enabling automated deployments to various environments such as staging and production.

# Key Concepts:

# Job:
A job is a single task that GitLab CI/CD executes as part of a pipeline. Jobs are defined in the `.gitlab-ci.yml` file and can include tasks such as building code, running tests, and deploying applications. Each job runs in a separate environment, typically within a Docker container or a virtual machine.

# Script:
The `script` keyword in GitLab CI/CD defines the commands to be executed for a specific job. These commands can include anything from compiling code to running tests or deploying applications. The script is written in YAML format and is executed within the environment specified for the job.

# Stage:
A stage is a logical grouping of jobs within a pipeline. Stages allow you to organize and visualize the progression of tasks from build to deployment. By defining stages in the `.gitlab-ci.yml` file, you can control the order in which jobs are executed and ensure that certain tasks are completed before others begin.

In GitLab CI/CD, variables are used to store information that can be used across different stages or jobs in a pipeline. These variables can be predefined within the GitLab project settings or defined dynamically within the CI/CD configuration files (e.g., `.gitlab-ci.yml`). 

# Variables can be of various types:

1. Global Variables: These are defined at the instance or group level and are available to all projects within that instance or group.

2. Project Variables: These are specific to a particular project and can be defined in the project settings. They are scoped to that project and can be used in its CI/CD pipelines.

3. Job Variables: These are defined within a specific job in the CI/CD configuration file and are only available within that job.


# CI/CD Workflow:

1. Commit: Developers commit code changes to the Git repository hosted on GitLab.

2. CI Pipeline Trigger: GitLab CI detects the new commit and triggers the CI pipeline defined in the `.gitlab-ci.yml` file.

3. Jobs Execution: GitLab CI executes the defined jobs in the pipeline, such as building code, running tests, and creating artifacts.

4. Stages: Jobs are organized into stages, ensuring that tasks progress in a logical sequence from build to deployment.

5. Testing: Automated tests are run to validate the code changes and ensure they meet quality standards.

6. Deployment: If all tests pass successfully, GitLab CD can automatically deploy the application to production or other environments.

By leveraging GitLab’s CI/CD capabilities, teams can automate their software delivery processes, improve code quality, and accelerate time-to-market for their applications.

# Connect your Kubernetes cluster with Gitlab 

we'll explore the seamless integration between GitLab agents and Microk8s Kubernetes clusters, demonstrating how to deploy a pod, service, and secret using YAML files that reference the Docker image we've built and stored in GitLab's container registry.

Firstly, we'll establish connectivity between GitLab and our Microk8s Kubernetes cluster by configuring a GitLab agent. This agent serves as the bridge between GitLab's CI/CD pipelines and our Kubernetes infrastructure, enabling automated deployment and management of containerized applications.

# Agent Access Token 

```
glagent-mFZRcD-orADRvCUJyCp3MLgfWQUyocGRczyt3Hdwrqrrx-2ikw
```

# Helm command to deploy Gitlab agent on your Kubernetes Cluster

```
helm repo add gitlab https://charts.gitlab.io
helm repo update
helm upgrade --install k8s-connection gitlab/gitlab-agent \
    --namespace gitlab-agent-k8s-connection \
    --create-namespace \
    --set image.tag=v17.0.0-rc4 \
    --set config.token=glagent-mFZRcD-orADRvCUJyCp3MLgfWQUyocGRczyt3Hdwrqrrx-2ikw \
    --set config.kasAddress=wss://kas.gitlab.com
```

# Let build docker image using job (pipeline) 

1. First login into the container registry in gitlab

docker login registry.gitlab.com  -u <Gitlab Username -p <GitlabPasswd >

2. Then Build the images using the html,css file to deploy website

docker build -t registry.gitlab.com/netcon2/<Project name> .

3 . Push the image to the container registry

docker push registry.gitlab.com/netcon2/<Project name>

# create a pipeline file as .git-ci.yml and copy & paste below commands

```
build_image:
stage: build
image: docker:20.10.16
services:
- docker:20.10.16-dind
before_script:
- docker login -u $REG_USER -p $REG_PASSWD $CI_REG
script:
- docker build -t $CI_REG/netcon2/k8s-data/web .
- docker push $CI_REG/netcon2/k8s-data/web
```

# Creating Kubernetes agent config.yaml in Gitlab

Creating a new <project name>/.gitlab/agents/<K8s-name>/config.yaml

```
ci_access:
  projects:
    - id: netcon2/k8s-data #Path_of_image&manifest_file_repo
```

# Variables

Variables are useful for storing sensitive information like credentials, API keys, or environment-specific configurations. They help in keeping sensitive data secure by not exposing them directly within the CI/CD configuration files. Additionally, they promote reusability and maintainability by allowing the same configuration to be used across different environments or branches.

# Add the deploy_server as stage in .gitlab-ci.yml to deploy the docker images in the kubernetes cluster(Microk8s)

```
variables:
KUBE_CONTEXT: netcon2/k8s:k8s-connection
CI_PROJECT_DIR: k8s

stages:
- build
- deploy

build_image:
stage: build
image: docker:20.10.16
services:
- docker:20.10.16-dind
before_script:
- docker login -u $REG_USER -p $REG_PASSWD $CI_REG
script:
- docker build -t $CI_REG/netcon2/k8s-data/web .
- docker push $CI_REG/netcon2/k8s-data/web

deploy_server:
stage: deploy
image:
name: bitnami/kubectl:latest
entrypoint: ['']
script:
- kubectl config use-context $KUBE_CONTEXT
- kubectl get pods
- kubectl get node -o wide
- kubectl apply -f $CI_PROJECT_DIR/svc.yaml
- kubectl apply -f $CI_PROJECT_DIR/secret.yaml
- kubectl apply -f $CI_PROJECT_DIR/pod.yaml
- kubectl get pod
- kubectl get svc
```

Next, armed with YAML files defining the desired state of our Kubernetes resources, we'll deploy a pod to encapsulate our application, a service to expose it internally or externally, and a secret to securely store sensitive information such as passwords or API keys. These YAML files will reference the Docker image stored in GitLab's container registry, ensuring that the latest version of our application is deployed consistently across our Kubernetes environment.

# To obtain the Manifest 

# Secret Manifest

```
microk8s kubectl create secret docker-registry Netflix --docker-server=registry.gitlab.com --docker-username='asdfghjkl' --docker-password='qwertyuiop' --dry-run=client -o yaml > secret.yaml

microk8s kubectl apply -f secret.yaml <to check whether is able to deploy in K8S cluster, this step will be done in CI/CD, don't do it>
```

# Pod Manifest file

```
microk8s kubectl run Netflix --image=registry.gitlab.com/netcon2/k8s-data/web --dry-run=client -o yaml > pod.yaml

microk8s kubectl apply -f pod.yaml <to check whether is able to deploy in K8S cluster, this step will be done in CI/CD, don't do it>
```

# Service Manifest file

```
microk8s kubectl create svc nodeport netflix-web --tcp=80:80 --dry-run=client -o yaml > svc.yaml 

microk8s kubectl apply -f svc.yaml <to check whether is able to deploy in K8S cluster, this step will be done in CI/CD, don't do it>
```

# Upload these Manifest file to the gitlab repo in a new directory (k8s)

By leveraging GitLab's robust CI/CD capabilities in tandem with Microk8s Kubernetes clusters, we can streamline the deployment process, enhance scalability, and maintain agility in our development workflows. 

```
deploy_server:
stage: deploy
image:
name: bitnami/kubectl:latest
entrypoint: ['']
script:
- kubectl config use-context $KUBE_CONTEXT
- kubectl get pods
- kubectl get node -o wide
- kubectl apply -f $CI_PROJECT_DIR/svc.yaml
- kubectl apply -f $CI_PROJECT_DIR/secret.yaml
- kubectl apply -f $CI_PROJECT_DIR/pod.yaml
- kubectl get pod
- kubectl get svc
```

Join us on this journey as we dive into the intricacies of deploying containerized applications to Kubernetes using GitLab agents and YAML configuration files, empowering you to harness the full potential of modern DevOps practices.

