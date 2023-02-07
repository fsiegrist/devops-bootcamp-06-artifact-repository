## Notes on the videos
<br />

<details>
<summary>Video: Install and Run Nexus on a Cloud Server</summary>
<br />

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
- `ps aux | nexus` shows the PID
- `netstat -tlnp` shows that the process with the nexus PID is listening on port 8081

So go to the DigitalOcean admin webpage and add a firewall rule opening the port 8081 for all IP addresses.

Now you can access Nexus in your browser via `http://<droplet-ip-address>:8081`.

</details>

*****

<details>
<summary>Video: Introduction to Nexus</summary>
<br />

There is a predefined `admin` user. Its password is stored in `/opt/sonatype-work/nexus3/admin.password`.
You can log in with this password on the nexus website. You will be prompted to change the password. As soon as you've changed it, the file `/opt/sonatype-work/nexus3/admin.password` will be removed.

</details>

*****

<details>
<summary>Video: Repository Types</summary>
<br />

Log in as admin user and click on the green settings button at the top of the page.
Click on Repositories. Several repositories are already predefined.

Repository Types:
- Proxy: Repository that is linked to a remote repository, e.g. maven-central. Nexus acts as a cache between the client retrieving an artifact and the linked remote repository.
- Hosted: Pimary storage to push your artifacts and components to, e.g. maven-releases, maven-snapshots. They have integrated version policies for the specific type, e.g. release-versions or snapshot-versions. This type can also be used to store third-party-libraries, that are not available in public repositories.
- Group: Multiple repositories of different types can be grouped together making them accessible via one single endpoint.

</details>

*****

<details>
<summary>Video: Publish Artifact to Repository</summary>
<br />

In order to allow Maven or Gradle to upload build artifacts to the Nexus repository, we have to create a Nexus user and give it the resquired privileges.

**Create a Nexus User:**

Click on the settings button, open the Security section on the left an click on Users and then on the button "Create local user". Fill in the form. Choose the nx-anonymous role for the moment.

Now click on Roles on the left side and then on the button "Create role" in the top right corner. Now you can grant the least amount of privileges a user with this role must have to fullfill its tasks. The view-privileges are for browsing, adding, deleting etc. artifacts. The privilege nx-repository-view-maven2-*-* allows all actions on all maven-repositories (proxy, hosted). Add the requires privileges to the role and save it.

Now go back to the users and open the just created user. Assign it the new role and remove the role nx-anonymous.

**Configure Gradle for Nexus:**

See [Gradle Documentation](https://docs.gradle.org/7.6/userguide/publishing_setup.html)

Add the following to the build.gradle file:

```groovy
plugins {
    id 'java-library'
    id 'maven-publish'
}

version '1.0.0-SNAPSHOT'

publishing {
    publications { // what do we want to publish
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version" + ".jar") {
                extension 'jar'
            }
        }
    }
    repositories { // where do we want to publish to
        maven {
            name 'nexus'
            def releasesRepoUrl = 'http://139.59.136.189:8081/repository/maven-releases/'
            def snapshotsRepoUrl = 'http://139.59.136.189:8081/repository/maven-snapshots/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            allowInsecureProtocol = true       // because we use http
            credentials {
                username project.repoUser      // to be defined in gradle.properties
                password project.repoPassword  // to be defined in gradle.properties
            }
        }
    }
}
```

To publish the artifact to the Nexus repository (after having built it using `gradlew build`), just execute the command `gradlew publish`.

**Configure Maven for Nexus:**

Add the following to the pom.xml file:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <version>2.7</version>
        </plugin>
    </plugins>
</build>

<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://139.59.136.189:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://139.59.136.189:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

The credentials are defined in the file ~/.m2/settings.xml:

```xml
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>...</username>
            <password>...</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>...</username>
            <password>...</password>
        </server>
    </servers>
</settings>
```

To publish the artifact to the Nexus repository (after having built it using `mvn package`), just execute the command `mvn deploy`. Setting the version to a SNAPSHOT version or to a release version will be enough to make maven deploy the artifact to the snapshot repository or to the release repository respectively.

</details>

*****

<details>
<summary>Video: Nexus REST API</summary>
<br />

Detailed information can be found in the [Nexus 3 Documentation](https://help.sonatype.com/repomanager3/integrations/rest-and-integration-api).

All REST API calls require the credentials of a user who has been granted the appropriate privileges in the Nexus UI to be passed along.

Get all available repositories:\
`curl -u user:pwd -X GET 'http://139.59.136.189:8081/service/rest/v1/repositories'`

Get all artifacts within a specific repository:\
`curl -u user:pwd -X GET 'http://139.59.136.189:8081/service/rest/v1/components?repository=maven-snapshots'`

Get all assets (files) of a specific component:\
`curl -u user:pwd -X GET 'http://139.59.136.189:8081/service/rest/v1/components/<component-id>'`

</details>

*****

<details>
<summary>Video: Blob Store</summary>
<br />

Detailed information can be found in the [Nexus 3 Documentation](https://help.sonatype.com/repomanager3/nexus-repository-administration/repository-management/configuring-blob-stores).

Blob stores are the store system of Nexus to store all the uploaded binary files. They can use the local file system or a cloud storage. Each blob store can be used by one or multiple repositories or repository groups.

Blob stores using the local file system can be found in the folder `/opt/sonatype-work/nexus3/blobs`.

When Nexus is installed there is one blob store created called `default`. New blob stores can be created in the Nexus UI. Be aware that blob stores can't be modified (e.g. give more space) and as long as a blob store is used by at least one repository it can't be deleted. A repository cannot be split over multiple blob stores. And once a repository is assigned to a blob store, you cannot assing it to another blob store. So when creating a new blob store, you have to carefully consider how to configure it.

</details>

*****

<details>
<summary>Video: Components vs Assets</summary>
<br />

Each component (java-app) consists of one or more assets (java-app-1.0-20230204.231003-10.jar, java-app-1.0-20230204.231003-10.pom, java-app-1.0-20230204.231003-10.pom.sha512, etc.).

In JAR repositories each component has its own assets. Assets belong to exactly one component version. In Docker repositories however, assets are given a unique ID and can be shared among different components, because components represent Docker images and assets represent image layers.

</details>

*****

<details>
<summary>Video: Cleanup Policies and Scheduled Tasks</summary>
<br />

Detailed information can be found in the Nexus 3 documentation on [Cleanup Policies](https://help.sonatype.com/repomanager3/nexus-repository-administration/repository-management/cleanup-policies) and [Tasks](https://help.sonatype.com/repomanager3/nexus-repository-administration/tasks).

Cleanup policies can be defined to cleanup no longer used components. When a policy has been defined, it can be previewed to check, whether the defined rules to not accidently delete components.

Once the policy definition is ok and the policy is saved, it has to be associated to a repository in order to become effective. This is done in the repository configuration section.

The policies are run by a scheduled task (cleanup service) and just mark the components to be deleted. The components won't be visible in the repository but the blob store still contains the data. In order to delete the assets physically from the disk, you have to create a scheduled task of type "Admin - Compact blob store".

To test scheduled tasks, they can be executed manually.

</details>

*****