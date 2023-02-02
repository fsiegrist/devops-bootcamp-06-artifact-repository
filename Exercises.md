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


</details>

******

<details>
<summary>Exercise 2: Create npm hosted repository</summary>
<br />

**Tasks:**

For a Node application you:
- create a new npm hosted repository with a new blob store

**Steps to solve the tasks:**


</details>

******

<details>
<summary>Exercise 3: Create user for team 1</summary>
<br />

**Tasks:**

- You create Nexus user for the project 1 team to have access to this npm repository

**Steps to solve the tasks:**

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

</details>

******

<details>
<summary>Exercise 5: Create maven hosted repository</summary>
<br />

**Tasks:**

For a Java application you:
- create a new maven hosted repository

**Steps to solve the tasks:**

</details>

******

<details>
<summary>Exercise 6: Create user for team 2</summary>
<br />

**Tasks:**

- You create a Nexus user for project 2 team to have access to this maven repository

**Steps to solve the tasks:**

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

</details>

******

<details>
<summary>Exercise 9: Download from Nexus and start application</summary>
<br />

**Tasks:**

You decide to automate the fetching from Nexus and starting the application So you:
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

</details>

******
