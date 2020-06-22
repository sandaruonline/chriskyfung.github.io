---
layout: post
title:  "QLogbook: Deploy to Kubernetes in Google Cloud: Challenge Lab"
author: chris
date: 2020-05-04 +0800
last_modified_at: 2020-06-22 10:15:00 +0800
category: Cloud
tags: [Qwiklabs, Google Cloud, Logbook]
permalink: /blog/qwiklabs/Deploy-to-Kubernetes-in-Google-Cloud-Challenge-Lab
redirect_from: /blog/qwiklabs/Kubernetes-in-Google-Cloud-Challenge-Lab
excerpt: A brief procedure for qwiklab GSP318 "Deploy to Kubernetes in Google Cloud&#58; Challenge Lab". It includes&#58; How to create Docker images, Deploy and update the containers to Kubernetes, and Create a pipeline in Jenkins.
image: 
   path: qwiklabs/qwiklab-GSP318-Multibranch_Pipeline
   ext: png
---

In this article, we will go through the lab **GSP318** _[Deploy to Kubernetes in Google Cloud: Challenge Lab](https://www.qwiklabs.com/focuses/10457?parent=catalog)_, which is labeled as an expert-level exercise (formerly known as _Kubernetes in Google Cloud: Challenge Lab_). You will practice the skills and knowledge for configuring Docker images and containers, and deploying fully-fledged Kubernetes Engine applications.

**The challenge contains 4 required tasks:**

1. Create a Docker image and store the Dockerfile
2. Test the created Docker image
3. Push the Docker image in the Google Container Repository
4. Create and expose a deployment in Kubernetes
5. Increase the replicas from 1 to 3
6. Update the deployment with a new version of valkyrie-app
7. Create a pipeline in Jenkins to deploy your app

## Task 1: Create a Docker image and store the Dockerfile

_Hint_: Refer procedures and modify the codes in the lab GSP055 [Introduction to Docker](https://google.qwiklabs.com/focuses/1029?parent=catalog#step5)

1. First of all, you have to run the following command in Cloud Shell.

   ```bash
   source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking.sh)
   ```

   It installs the marking scripts, which use to check your progress.

2. Then, run the commands below to clone the valkyrie-app source code repository to the Cloud Shell. (_Remember to **replace**_ `YOUR_PROJECT` _with your Project ID_)

   ```bash
   gcloud source repos clone valkyrie-app --project=YOUR_PROJECT
   ```

3. Create a `Dockerfile` under the `valkyrie-app` directory and add the configuration to the file. Copy the given codes from the lab page to the following snippet, and then run the commands in the Cloud Shell.

   ```bash
   cd valkyrie-app

   cat > Dockerfile <<EOF
   
   // COPY TO HERE
   
   EOF
   ```

4. Build the image with the following command:

   ```bash
   docker build -t valkyrie-app:v0.0.1 .
   ```

5. Run `docker images` to look at the images you built.

Before clicking **Check my progress** in the lab page, don't forget to run the following commands to execute the marking script:

{:.ml-3}
```bash
cd ~/marking
./step1.sh
```

## Task 2: Test the created Docker image

_Hint_: Refer procedures and modify the codes in the lab GSP055 [Introduction to Docker](https://google.qwiklabs.com/focuses/1029?parent=catalog#step6)

The lab instruction requires you to run the docker image built in task 1 and show the running application by **Web Preview** on port 8080. Based on the requirements, the docker command will be:

{:.ml-3}
```bash
docker run -p 8080:8080 --name valkyrie-app valkyrie-app:v0.0.1 &
```

1. In the Cloud Shell, go back to the `valkyrie-app` directory, and run the above command.

2. Click **Web Preview** to see the running app.

   {% include picture.html img="qwiklabs/qwiklab-GSP318-valkyrie-app-v0.0.1" ext="png" alt="" class="ml-4" %}

After that, open a new Cloud Shell to run the `step2.sh` marking script.

{:.ml-3}
```bash
cd ~/marking
./step2.sh
```

## Task 3: Push the Docker image in the Container Repository

_Hint_: Refer procedures and modify the codes in the lab GSP055 [Introduction to Docker](https://google.qwiklabs.com/focuses/1029?parent=catalog#step8)

In this task, you will push the Docker image valkyrie-app:v0.0.1 into the Container Registry with a tag `gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.1`.

Thus, you should format the docker commands as below.
(_Remember to **replace**_ `YOUR_PROJECT` _with your Project ID_)

```bash
docker tag valkyrie-app:v0.0.1 gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.1
docker images
docker push gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.1
```

After pushing the container, the `valkyrie-app` repository will appear in the Cloud Console as shown in the image below.

{% include picture.html img="qwiklabs/qwiklab-GSP318-container-repositories" ext="png" alt="Push the Docker image of valkyrie-app in the Google Container Repository" %}

## Task 4: Create and expose a deployment in Kubernetes

_Hint_: Refer procedures in the labs GSP100 [Kubernetes Engine: Qwik Start](https://google.qwiklabs.com/focuses/878?parent=catalog) and GSP021 [Orchestrating the Cloud with Kubernetes](https://google.qwiklabs.com/focuses/557?parent=catalog) for steps 1-2 and steps 3-4, respectively.

1. In the Cloud Shell, go to the `valkyrie-app/k8s` subdirectory.

2. Get authentication credentials for the cluster

   ```bash
   gcloud container clusters get-credentials valkyrie-dev
   ```

3. Use a text editor to modify `deployment.yaml` and replace `IMAGE_HERE` with `gcr.io/YOUR-PROJECT-ID/valkyrie-app:v0.0.1`

4. Use `kubectl create -f` command to deploy **deployment.yaml** and **service.yaml**

## Task 5: Update the deployment with a new version of valkyrie-app

_Hint_: Refer the skills in lab GSP053 [Managing Deployments Using Kubernetes Engine](https://google.qwiklabs.com/focuses/639?parent=catalog) or my previous article [Qwiklabs/Logbook: Scale Out and Update a Containerized Application on a Kubernetes Cluster](/blog/qwiklabs/Scale-Out-and-Update-a-Containerized-Application-on-a-Kubernetes-Cluster)

**Step 5-1** Increase the replicas from 1 to 3

**Step 5-2** Update the deployment with a new version of valkyrie-app

1. Go back to the `valkyrie-app` directory in the Cloud Shell.

2. Merge the branch called `kurt-dev` into master using the following `git` command:

   ```bash
   git merge origin/kurt-dev
   ```

3. Build and push the new version with tagged v0.0.2:
(_Remember to **replace**_ `YOUR_PROJECT` _with your Project ID_)

   ```bash
   docker build -t valkyrie-app:v0.0.2 .
   docker tag valkyrie-app:v0.0.2 gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.2
   docker images
   docker push gcr.io/YOUR_PROJECT/valkyrie-app:v0.0.2
   ```

4. Trigger a rolling update by running the following command:

   ```bash
   kubectl edit deployment valkyrie-app
   ```

   Change the image tag from `v0.0.1` to `v0.0.2`. then save and exit.

## Task 6: Create a pipeline in Jenkins to deploy your app

_Hint_: Refer procedures in the labs GSP051 [Continuous Delivery with Jenkins in Kubernetes Engine](https://google.qwiklabs.com/focuses/1104?parent=catalog)

In this task, you will need to:

1. Connect to Jenkins
2. Adding your service account credentials
3. Creating a Jenkins job
4. Modifying the pipeline definition
5. Modify the site
6. Kick off Deployment

#### 6.1 To connect the Jenkines

1. Get the password with the following command:

   ```bash
   printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
   ```

2. Connect to the Jenkins console using the commands below:

   ```
   export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
   kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
   ```

   If there is another running container, use the `docker` commands below to kill it:

   ```bash
   docker ps
   docker container kill $(docker ps -q)
   ```

3. Click on the **Web Preview** button in cloud shell, then click “Preview on port 8080” to connect to the Jenkins console.<br>
   **Username**: admin

#### 6.2 Adding your service account credentials

1. In the Jenkins user interface, click **Credentials** in the left navigation.

2. Click **Jenkins**

3. Click **Global credentials (unrestricted)**.

4. Click **Add Credentials** in the left navigation.

5. Select **Google Service Account from metadata** from the **Kind** drop-down and click **OK**.

#### 6.3 Creating the Jenkins job

Create a pipeline job that points to your */master branch on your source code.

1. Click **Jenkins** > **New Item** in the left navigation:

2. Name the project **valkyrie-app**, then choose the **Multibranch Pipeline** option and click **OK**.

3. On the next page, in the Branch Sources section, click **Add Source** and select **git**.

4. Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories `https://source.developers.google.com/p/YOUR_PROJECT/r/valkyrie-app` into the Project Repository field. Remember to replace **YOUR_PROJECT** with your GCP Project ID.

5. From the **Credentials** drop-down, select the name of the credentials you created when adding your service account in the previous steps.

6. Under **Scan Multibranch Pipeline Triggers** section, check the **Periodically if not otherwise run** box and set the **Interval** value to 1 minute.

7. Your job configuration should look like this:

   {% include picture.html img="qwiklabs/qwiklab-GSP318-Multibranch_Pipeline" ext="png" alt="" class="ml-4" %}

#### 6.4 Modifying the pipeline definition

Open `Jenkinsfile` file in a text editor, and replace **YOUR_PROJECT** with your GCP project ID.

#### 6.5 Modify the site

Open `source/html.go` file in a text editor, and change the color of headings from green to orange.

#### 6.6 Kick off Deployment

Commit and push the changes:

```bash
git config --global user.email YOUR_PROJECT
git config --global user.name YOUR_PROJECT

git add Jenkinsfile source/html.go
git commit -m 'green to orange'
git push origin master
```

Finally, manually trigger the build in the Jenkins console

{% include picture.html img="qwiklabs/qwiklab-GSP318-jenkins-build-queue" ext="png" alt="" %}

{% include picture.html img="qwiklabs/qwiklab-GSP318-valkyrie-app-dev.2" ext="png" alt="" %}

## Summary

Do you feel this challenge lab quite difficult? You might if you don't get enough practice on Kubernetes Engines. Other than that, you might be disturbed if you don't know how to use `docker` command to kill a container and `git config` command before committing the changes. Despite this, most tasks reproduce based on the contents from the labs in the Kubernetes in Google Cloud quest. There is nothing to be afraid of.

* * *

**See Also**: [Learning Google Cloud Platform on Qwiklabs: Learning Map, Assistive Tool and Tips](/blog/qwiklabs/Qwiklabs-User-Tips-for-Learning_Google_Cloud_Platform)