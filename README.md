## **Jenkins and its stages**

Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation software DevOps tool; It is used to implement CI/CD workflows, called pipelines.

As well as Jenkins, there are also other automation servers such as Bamboo and TeamCity. These automation servers all share similar capabilities however Jenkins’ highlights are its community and many plugins whilst Bamboo and TeamCity focus on product integrations such as Jira and IntelliJ.

A stage (or stage block) is a specific part of a task performed through the pipeline; for example, build, test, deploy.

![](images/stages.PNG)

## **Jenkins process flows**

The Jenkins process initally begins with all team members pushing tested code to a single repository; Continuous Integration is the practice of automating the integration of code continuously from all team members.

Jenkins can be set up to automatically test code and transport it to the master node, if all test cases past; or else, they are given to the agent node for feedback and review to the relevant team members.

When tested code is ready, it can be deployed at any point through manual releases; this is defined as Continuous Delivery. This step can also be automated to enable Continuous Deployment.

![](images/process.PNG)

## **Accessing a Jenkins server and checking its OS**

1. Navigate to a Jenkins server, typically available on port `8080` and login in with the suitable credentials.

![](images/login.PNG)

2. Click on the `New Item` tab on the left panel and Create `Freestyle project`.

![](images/freestyle.PNG)

3. Firstly, discard old builds and keep a maximum of 3 builds then navigate to the Build section and select "execute shell". Check the os using command "uname -a".

![](images/uname.PNG)

4. Navigate to the "build history".

![](images/build.PNG)

5. Read the Console Output under the drop down menu to show the OS and its version.

![](images/console.PNG)


## **Section 1: Testing an application using Jenkins**

1. Follow steps 1 - 3 as shown above and have a GitHub repository ready with the application stored.

2. Add a GitHub project using the HTTPS URL from the repository.

![](images/https.PNG)

3. Select the `Restrict where this project can be run` option to restrict the build when not in use. A label expression has already been provided, enter one as required.

> Note: This example build is only used to test the application.

![](images/restrict.PNG)

4. Add the version control system using the github repository's SSH URL. Create a SSH key within the GitHub repository (`Settings` > `Deploy keys`) allowing read/write access and save it to the local `.ssh` folder. Use this SSH key as the `credentials` with the option SSH username with private key.

>Note: Specify the branch as `*/main` for this example; this will need to be amended when a new branch is introduced (i.e. `*/dev`) in the latter example.

![](images/ssh.PNG)

5. Tick the `GitHub hook trigger for GITScm polling` option to trigger a build any time a change is made in the GitHub repository; the repository is also shown within the Jenkins `workspace` for the particular item.

![](images/webhook.PNG)

6. `Provide Node & npm bin/ folder to PATH` to attach the required dependencies and environment variable.

![](images/env.PNG)

7. Execute a script on build to navigate to the correct file path to install npm and its testing module.

![](images/shell.PNG)

8. The `Console Output` should then display the following with all tests passed.

![](images/test.PNG)

## **Section 2: Configure the webhook integration between Jenkins and GitHub**

1. Navigate to `settings` and `webhooks` within the GitHub repository and `add webhook`.

2. Enter the payload URL in the format `http://<public-ipv4-address>:8080/github-webhook/` and select the content type as `application/json`.

![](images/webhook-setup.PNG)

3. No secret is required, select `just the push event` and `active` then proceed to `add webhook`.

![](images/pushevent.PNG)

4. The build history should then update and trigger a test.

![](images/webhook-done.PNG)

## **Section 3: Creating an automated Jenkins CI merge item**

This section ties in a new branch called `dev` which is to be merged to the `main` branch in the GitHub repository when a `push` is initiated.

1. First, create a new branch in the git repository and call it `dev`.

```bash
git branch dev # Create a branch without moving to it.

git checkout dev # Move to the branch.
```

> Note: the new branch may need to be initialised through pushing it to the github repo.

2. Create a new item in Jenkins called `name-ci-merge`.

3. Configure your same GitHub repository endpoints firstly with HTTPS then with SSH for Source Code Management, as shown in section 1 steps 1 - 4.

4. Specify the branch as the new branch `*/dev` and add the plugin (or behaviour) `Merge before build`, save the item; ensure that you are merging the branch to main as shown below.

![](images/merge-scm.PNG)

5. Return to the original CI item and change the branch specifier to `*/dev` as well.

![](images/ci-dev.PNG)

6. Add the CI merge item to the post-build actions if the tests pass to automate the trigger.

![](images/ci-post.PNG)

7. Add the `post-build action` to push the changes made on the `dev` branch to the GitHub `main` repository.

![](images/publisher.PNG)


## **Section 4: Configuring Continuous Delivery (CD) between Jenkins and AWS**

1. Create a new item called `name-cd` and configure it as usual with the github project and source code management (as shown in section 1 steps 1 - 4), but this time with `*/main` as the branch to build.

2. Provide the SSH agent PEM file to access AWS.

![](images/build-pem.PNG)

3. Include an execute shell to copy the over the github `main` repository, connecting to the instance through ssh whilst removing the requirement to provide a key, execute the provision file and start the application.

> Note: If the application is not updating, this could be due to the user changing to the incorrect app directory.

```bash
rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<Public IPv4 DNS>:/home/ubuntu
ssh -o "StrictHostKeyChecking=no" ubuntu@<Public IPv4 DNS> <<EOF
	sudo bash ./app/app/provsion.sh
	cd app/app/app
	pm2 kill
	pm2 start app.js
EOF
```

![](images/cd-build.PNG)

4. Add this job to trigger at the end of the `<name>-merge-ci` build as a `post-build action`.

5. To test continuous delivery is in operation, change the HTML file through the `dev` branch. Push this change to the GitHub repository and the pipeline build should trigger. Navigate to the web browser and enter the IPv4 address.

![](images/final.PNG)

## **Section 5: Building a Jenkins server**

1. Create a new virtual machine with AWS EC2 configured with ubuntu 18.04 LTS and a t2.micro. Additionally, set the inbound rules to allow SSH access from `my IP` as well as HTTP and custom TCP, jenkins port 8080, from anywhere. Launch this instance and when ready, SSH into it update the APT package manager.

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
```

2. Add the key to your system to use this repository.

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

3. Add a Jenkins apt repository entry.

```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

4. Update your local package index then install Java and Jenkins. Finally, start Jenkins through the system process.

```bash
sudo apt-get update

sudo apt-get install fontconfig openjdk-11-jre -y

sudo apt-get install jenkins

sudo systemctl start jenkins
```

5. Access the interface of the automation server on the web browser through `http://<public-ip4-address>:8080/`. The user will be prompted to enter the admin password, which can be found in the `initialAdminPassword` file.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

6. Select what plugins are to be installed and tick the following.

![](images/jenkins-install-plugins.PNG)

![](images/jenkins-install-plugins-node.PNG)

![](images/jenkins-install-plugins-ssh.PNG)

![](images/jenkins-install-plugins-github.PNG)

7. The user will now be presented with the updating page, which may take some time to complete. Further credential updates are required, including usernames and passwords to access the server.

![](images/jenkins-install-plugins-update.PNG)

8. Once within Jenkins, navigate to the `Dashboard`. On the side panel, select `Manage Jenkins` then `Security`. Configure the Git Host Key settings to `Accept first connection` for first key verification.

![](images/jenkins-security-git.PNG)

9. Also, within `Manage Jenkins` and under `Tools`, include the installation for version 12 for NodeJS.

10. To test the automation server, follow sections 1 - 4 to setup a pipeline with a webhook to a GitHub repository and create a new app AMI instance.