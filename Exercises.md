You and other teams in 2 different projects in your company see at some point, that they have many different small projects, the NodeJS application you built in the previous step, the java-gradle helper application and so on. So you discuss and decide it will be good to be able to keep all these app artifacts in 1 place, where each team can keep their app artifacts and can access them when they need.

So they ask you to setup Nexus in the company and create repositories for 2 different projects.

<details>
<summary>Exercise 0: Create Git Repository</summary>
<br />

**Tasks:**

Create a git repository for the module exercises on GitHub.

**Steps to solve the tasks:**

```sh
# create a local repository and commit its content
mkdir devops-bootcamp-06-artifact-repository
cd devops-bootcamp-06-artifact-repository
touch README.md
touch Notes.md
touch Exercises.md
git init 
git add .
git commit -m "Initial commit"

# create git repository on GitHub and push your newly created local repository to it
git remote add origin git@github.com:fsiegrist/devops-bootcamp-06-artifact-repository.git
# rename master branch to main if necessary (default on GitHub)
git branch -M main
# push your newly created local repository to it
git push -u origin main
```

</details>

******

<details>
<summary>Exercise 1: Install Nexus on a server</summary>
<br />

**Tasks:**

If you already followed the demo in the Nexus module for installing Nexus, then you can use that one. If not, you can watch the module demo video to install Nexus. 

**Steps to solve the tasks:**

Login to your account on [DigitalOcean](https://cloud.digitalocean.com/login) and create a new Droplet having at least 4GB RAM (better 8GB). Create a firewall rule opening the ports 22 for SSH.

SSH into the server:
- copy the Droplet's IP address
- execute `ssh root@<droplet-ip-address>`

Install Java Version 8 (needed for Nexus):
- `apt update`
- `apt install openjdk-8-jre-headless`

Download and unpack the latest Nexus version into the /opt folder:
- `cd /opt`
- `wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz`
- `tar -zxvf latest-unix.tar.gz` => two folders nexus-3.46.0-01 and sonatype-work

Create a nexus user to be used to run the nexus application:
- `adduser nexus`

Change the privileges for the unpacked folders (nexus user needs to access both):
- `chown -R nexus:nexus nexus-3.46.0-01`
- `chown -R nexus:nexus sonatype-work`

Configure Nexus to run with the nexus user we just created:
- Add `run_as_user="nexus"` to the file `nexus-3.46.0-01/bin/nexus.rc` using vim

Switch to the nexus user and start Nexus:
- `su - nexus`
- `/opt/nexus-3.46.0-01/bin/nexus start`

Check the port on which Nexus is running:
- `ps aux | grep nexus` shows the PID
- `netstat -tlnp` shows that the process with the nexus PID is listening on port 8081

So go to the DigitalOcean admin webpage and add a firewall rule opening the port 8081 for all IP addresses.

Open Nexus in your browser via `http://<droplet-ip-address>:8081`.

</details>

******

<details>
<summary>Exercise 2: Create npm hosted repository</summary>
<br />

**Tasks:**

For a Node application you:
- create a new npm hosted repository with a new blob store

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

Create a new blob store:
- Go to Settings > Repository > Blob Stores
- Press the blue 'Create Blob Store' button
- Choose type 'File'; enter a name for the new blob store (e.g. devops-bootcamp); press the 'Save' button

Create a new npm hosted repository:
- Go to Settings > Repository > Repositories and press the 'Create repository' button
- Select the type/recipe 'npm (hosted)'
- Enter a name for the new repository (e.g. npm)
- Select the blob store created before ('devops-bootcamp')
- Leave all other form fields unchanged and press the blue 'Create repository' button

</details>

******

<details>
<summary>Exercise 3: Create user for team 1</summary>
<br />

**Tasks:**

- You create Nexus user for the project 1 team to have access to this npm repository

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

Create a new user:
- Go to Settings > Security > Users
- Press the 'Create local user' button
- Enter an ID (e.g. project-1); fill in all other mandatory form fields; select the status 'Active'; assign the role 'nx-anonymous'
- Press the blue 'Create local user' button

Create a new role:
- Go to Settings > Security > Roles and press the 'Create Role' button
- Choose the type 'Nexus role'; enter an id (e.g. nx-npm) and a name;
- Select the privilege 'nx-repository-view-npm-*-*'
- Press the 'Save' button

Assign the new role to the new user:
- Go back to Settings > Security > Users and select the new project-1 user
- Move the new role 'nx-npm' to the granted roles and remove the previously assigned role 'nx-anonymous'
- Press the 'Save' button

</details>

******

<details>
<summary>Exercise 4: Build and publish npm tar</summary>
<br />

**Tasks:**

You want to test that the project 1 user has correct access configured. So you:
- build and publish a nodejs tar package to the npm repo

Use: Node application from Cloud & IaaS Basics exercises

Hint:

```sh
# for publishing project tar file 
npm publish --registry={npm-repo-url-in-nexus} {package-name}
```

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

To enable the npm publish feature, an additional realm has to be added. Go to Settings > Security > Realms, add the 'npm Bearer Token Realm' and save the changes.

Switch to your local machine and create a file ~/.npmrc with the following content:
```sh
registry=http://<droplet-ip-address>:8081/repository/npm/
auth-type=legacy
```

Open a terminal and execute the following commands:

```sh
# switch to the directory of the node app in module 5
cd [/path/to/devops/bootcamp/module-5/git/repo]/app

# create the tar file to be pushed to the Nexus npm repository
npm install
npm pack

# login to your Nexus npm repository (registry)
npm login # enter username (project-1) and password (xxxxx) of the new user

# publish the tar file to the Nexus repository
npm publish bootcamp-node-project-1.0.0.tgz

# logout
npm logout
```

</details>

******

<details>
<summary>Exercise 5: Create maven hosted repository</summary>
<br />

**Tasks:**

For a Java application you:
- create a new maven hosted repository

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

Create a new maven hosted repository:
- Go to Settings > Repository > Repositories and press the 'Create repository' button
- Select the type/recipe 'maven2 (hosted)'
- Enter a name for the new repository (e.g. maven-repo)
- Select version policy 'Snapshot'
- Select the blob store created in exercise 2 ('devops-bootcamp')
- Leave all other form fields unchanged and press the blue 'Create repository' button

</details>

******

<details>
<summary>Exercise 6: Create user for team 2</summary>
<br />

**Tasks:**

- You create a Nexus user for project 2 team to have access to this maven repository.

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

Create a new user:
- Go to Settings > Security > Users
- Press the 'Create local user' button
- Enter an ID (e.g. project-2); fill in all other mandatory form fields; select the status 'Active'; assign the role 'nx-anonymous'
- Press the blue 'Create local user' button

Create a new role:
- Go to Settings > Security > Roles and press the 'Create Role' button
- Choose the type 'Nexus role'; enter an id (e.g. nx-maven-repo) and a name;
- Select the privilege 'nx-repository-view-maven2-*-*'
- Press the 'Save' button

Assign the new role to the new user:
- Go back to Settings > Security > Users and select the new project-2 user
- Move the new role 'nx-maven-repo' to the granted roles and remove the previously assigned role 'nx-anonymous'
- Press the 'Save' button

</details>

******

<details>
<summary>Exercise 7: Build and publish jar file</summary>
<br />

**Tasks:**

You want to test that the project 2 user has the correct access configured and also upload the first version. So:
- build and publish the jar file to the new repository using the team 2 user.

_Use: Java-Gradle application from Build Tools exercises_

**Steps to solve the tasks:**

Open a terminal on your local machine and execute the following commands:

```sh
# switch to the directory of the java gradle app in module 4
cd [/path/to/devops/bootcamp/module-4/git/repo]/app

# open the file build.gradle and adjust it as described below
vim build.gradle
# 1. add `id 'maven-publish'` to the plugins block
# 2. append the following publishing block at the end of the file:
    publishing {
        publications {
            mavenJava(MavenPublication) {
                artifact("build/libs/bootcamp-java-project-$version" + ".jar") {
                    extension 'jar'
                }
            }
        }
        repositories {
            maven {
                name 'nexus'
                url = 'http://139.59.136.189:8081/repository/maven-repo/'
                allowInsecureProtocol = true
                credentials {
                    username project.repoUser
                    password project.repoPassword
                }
            }
        }
    }

# create a file ./gradle.properties and add the credentials for the project-2 user:
touch ./gradle.properties
echo 'repoUser=project-2' > gradle.properties
echo 'repoPassword=xxxx' >> gradle.properties

# change the gradle version to 7.6 (otherwise the maven-publish plugin does not work together with java version 17 installed on the local machine)
vim gradle/wrapper/gradle-wrapper.properties # replace gradle-7.0-bin.zip with gradle-7.6-bin.zip

# build the jar file to be pushed to the Nexus maven repository
./gradlew build

# publish the jar file in the build/libs directory to the Nexus maven repository
./gradlew publish
```

</details>

******

<details>
<summary>Exercise 8: Download from Nexus and start application</summary>
<br />

**Tasks:**

- Create new user for droplet server that has access to both repositories
- On a digital ocean droplet, using Nexus Rest API, fetch the download URL info for the latest NodeJS app artifact
- Execute a command to fetch the latest artifact itself with the download URL
- Untar it and run on the server!

Hint:

```sh
# fetch download URL with curl
curl -u {user}:{password} -X GET 'http://{nexus-ip}:8081/service/rest/v1/components?repository={node-repo}&sort=version'
```

**Steps to solve the tasks:**

Open Nexus in your browser via `http://<droplet-ip-address>:8081` and login as admin user.

Create a new user:
- Go to Settings > Security > Users
- Press the 'Create local user' button
- Enter an ID (e.g. all-repo-reader); fill in all other mandatory form fields; select the status 'Active'; assign the roles 'nx-maven-repo' and 'nx-npm'
- Press the blue 'Create local user' button

SSH into the droplet server created for module 5:\
`ssh root@<module-5-droplet-ip-address>`

Execute the following command (replace the placeholders in `<...>` with their respective values):\
`curl -u all-repo-reader:<all-repo-reader-password> -X GET 'http://<nexus-droplet-ip-address>:8081/service/rest/v1/components?repository=npm&sort=version'`

The output contains components with their assets. Copy the "downloadUrl" value of the required asset.

Execute one of the following commands (replace the placeholders in `<...>` with their respective values):\
`curl -u all-repo-reader:<all-repo-reader-password> -X GET '<download-url>' --output bootcamp-node-project-1.0.0.tgz`

or 

`wget --user=all-repo-reader --password=<all-repo-reader-password> <download-url>`

Unpack the downloaded tar file (z=unzip and x=untar):\
`tar zxvf bootcamp-node-project-1.0.0.tgz`

Unpacking the tar file leaves a folder called `package`. Execute the following commands to start the app:
```sh
cd package
# install dependencies
npm install
# run the application in detached mode
node server.js &
# the server now listens on port 3000 (ps aux | grep node; netstat -tlnp)
```

No you can open your browser and visit `http://<module-5-droplet-ip-address>:3000` to see the application in action.

</details>

******

<details>
<summary>Exercise 9: Automate</summary>
<br />

**Tasks:**

You decide to automate the fetching from Nexus and starting the application. So you:
- Write a script that fetches the latest version from npm repository. Untar it and run on the server!
- Execute the script on the droplet

Hint:

```sh
# save the artifact details in a json file
curl -u {user}:{password} -X GET 'http://{nexus-ip}:8081/service/rest/v1/components?repository={node-repo}&sort=version' | jq "." > artifact.json

# grab the download url from the saved artifact details using 'jq' json processor tool
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)

# fetch the artifact with the extracted download url using 'wget' tool
wget --http-user={user} --http-password={password} $artifactDownloadUrl
```

**Steps to solve the tasks:**

SSH into the droplet server created for module 5:\
`ssh root@<module-5-droplet-ip-address>`

Create a file called `download-and-start-nodejs-app.sh` with the following content (replace the variable values at the beginning with the values of your setup):

```sh
#!/bin/bash

# set variables
repo_user_name=all-repo-reader
repo_user_password=xxxx
nexus_droplet_ip_address=139.59.136.189
npm_repo_name=npm

# save the artifact details in a json file
curl -u ${repo_user_name}:${repo_user_password} -X GET "http://${nexus_droplet_ip_address}:8081/service/rest/v1/components?repository=${npm_repo_name}&sort=version" | jq . > components.json

# grab the download url from the saved component details using 'jq' json processor tool
# .items[-1] selects the last item
downloadUrl=$(jq .items[-1].assets[0].downloadUrl components.json --raw-output)

# fetch the artifact with the extracted download url using 'wget' tool
wget --http-user=${repo_user_name} --http-password=${repo_user_password} ${downloadUrl} -O bootcamp-node-project-latest.tgz

# unpack the downloaded tar file (z=unzip and x=untar)
tar zxvf bootcamp-node-project-latest.tgz

# switch to package directory
cd package

# install dependencies
npm install

# run the application in detached mode
node server.js &
```

Execute `chmod u+x download-and-start-nodejs-app.sh` and run the script.

</details>

******
