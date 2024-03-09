---
title: Azure Kubernetes Workflow Jobs
date: "2022-08-09T23:41:37.121Z"
template: "post"
draft: false
slug: "/azure/kubernetes-workflow-jobs"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Understanding jobs in deploytoAksCluster.yml"
---

I have been trying to deploy my project using Azure Kubernetes. When I created a Azure Kubernetes(AKS) cluster, it looked really easy to integrate my Github repository to implement CI/CD.
![selecting-github](https://i.imgur.com/GgQfBOp.png)
![integrating-github](https://i.imgur.com/k7wu6Rx.png)


It seemed successful. I got to see the Dockerfile I had written

![dockerfile-found](https://i.imgur.com/8e0zYmA.png)

The Github Action was successful.

![action-succeeded](https://i.imgur.com/Px3HgqJ.png)

and I got to see the Swagger Document of the FastAPI server that I built.

However, when I tried to access one of the APIs, I ran into this error. 

![environment-variable-error](https://i.imgur.com/sGOq5x3.png)

The container on the AKS cluster could not access the environment variables that I put in from Github Secrets. I am familiar with Github Actions and workflows. I know how to set up environment variables using `env` or `with` so that I can use Github Secrets by writing down `${{ secrets.MY_GITHUB_SECRET }}`. However, none of them worked.

So I decided to look into the workflow and see what each job is responsible for. 

When you connect an AKS cluster with a Github repository using the deployment center attribute, there are 7 jobs in `deploytoAksCluster.yml`. Let's go through it one by one and see where I could possibly put my Github Secrets.

1. actions/checkout@master
    - it clones the repository before building a container. 

2. azure/docker-login@v1
    - it logs into the docker server from azure.
    - the login info was automatically added to Github Secrets when I connected the repository to the AKS cluster
    - so this workflow can access Github Secrets

3. Build and push image to ACR
    - just like the name says, it builds a docker image from the Dockerfile that was in the repository
    - and then it pushed the image to an Azure container registry, which is also connected to the AKS cluster
    - but when you think about it, when you run a docker container, you either put your environment variables in your Dockerfile or pass them as arguments when you run the container. 
    - in this case, you can't put your environment varibles in the Dockerfile because the Dockerfile cannot access Github Secrets directly. otherwise, you would have to put your secrets into your repository, which is not acceptable. 
    - so you would now have to pass the environment varibles when you use the `docker run` command. because I tried to pass environment variables like below but failed
    ```bash
    docker build --build-arg KEY=${{ secrets.ENVIRONMENT_VARIABLE}} \
    "$GITHUB_WORKSPACE/" -f  "Dockerfile" -t $AZURE_CONTAINER_REGISTRY:${{ github.sha }} \
    --label dockerfile-path=Dockerfile
    ```

4. [azure/k8s-set-context@v1](https://github.com/Azure/k8s-set-context)
    - this action is used to set cluster context
    - a cluster context is a set of access parameters that contains a Kubernetes cluster, a user, and a namespace
    - I actually tried to put in environment varibles here but it throws an error like below
    ![set-context error](https://i.imgur.com/heepGFz.png)

5. Create namespace
    - just like the name says, it creates [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for the AKS.
    - there are multiple pods, which are like docker containers, in a namespace
    - it gives the Github repository I connected a unique namespace

6. [azure/k8s-create-secret@v1](https://github.com/Azure/k8s-create-secret)
    - what a name! it says create-secret, but don't worry about it. I tried to put in secrets here but failed miserably. I pretty much got the same error I got in `azure/k8s-set-context`.
    - it says you can create a generic secret or docker-resitry secret in the cluster context which was set earlier through `azure/k8s-set-context`
    - but this section has to be it. so I am gonna try to manipulate this section. I will post another blog if I succeed.
    - I put something like `GITHUB_SECRET` but maybe only `github-secret` works. 

7. [azure/k8s-deploy@v1.2](https://github.com/Azure/k8s-deploy) 
    - this one is kinda obvious. it deploys the built container to the AKS cluster to make it accesible.
    - this job has an input called `imagepullsecrets` where you can put a docker-registry secret that has already been set up with the cluster. 
    - and this docker-registry secret could be set in `azure/k8s-create-secret`

That's it. I will let you guys know if I succeed in a later post.