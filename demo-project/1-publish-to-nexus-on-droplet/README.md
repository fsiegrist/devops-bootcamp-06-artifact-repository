## Demo Project - Publish Artifact to Nexus Running on a DigitalOcean Server

### Topics of the Demo Project
Run Nexus on Droplet and Publish Artifact to Nexus

### Technologies Used
- Nexus
- DigitalOcean
- Linux
- Java
- Gradle
- Maven

### Project Description
- Install and configure Nexus from scratch on a cloud server
- Create new User on Nexus with relevant permissions
- Java Gradle Project: Build Jar & Upload to Nexus
- Java Maven Project: Build Jar & Upload to Nexus

#### Steps to install and configure Nexus from scratch on a cloud server
**Step 1:** Create a Droplet on DigitalOcean\
Login to your account on [DigitalOcean](https://cloud.digitalocean.com/login) and create a new Droplet having at least 4GB RAM (better 8GB). Create a firewall rule opening the ports 22 for SSH.

**Step 2:** Install Java and net-tools
```sh
# SSH into the server
ssh root@<droplet-ip-address>

# install Java version 8 (needed for Nexus) and net-tools (needed for the netstat command):
apt update
apt install openjdk-8-jre-headless
apt install net-tools
```

**Step 3:** Install Nexus
```sh
# download and unpack the latest Nexus version into the /opt folder
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
```

**Step 4:** Create nexus user
```sh
adduser nexus

# change the privileges for the unpacked folders (nexus user needs to access both):
chown -R nexus:nexus nexus-3.46.0-01
chown -R nexus:nexus sonatype-work
```

**Step 5:** Configure Nexus to run with the nexus user we just created\
Add `run_as_user="nexus"` to the file `nexus-3.46.0-01/bin/nexus.rc` using vim.

**Step 6:** Start Nexus
```sh
# switch to the nexus user and start Nexus
su - nexus
/opt/nexus-3.46.0-01/bin/nexus start

# check the port on which Nexus is running
ps aux | grep nexus # shows the PID of the nexus process
netstat -tlnp # shows that the process with the nexus PID is listening on port 8081
```

So go to the DigitalOcean admin webpage and add a firewall rule opening the port 8081 for all IP addresses.

#### Steps to create new user on Nexus with relevant permissions
**Step 1:** Login as admin user
- Open your browser and navigate to `http://<droplet-ip-address>:8081` to access the Nexus login page. 
- There is a predefined `admin` user. Its password is stored in `/opt/sonatype-work/nexus3/admin.password`. Log in with this password and change it. 
- Login again with the new password.

**Step 2:** Create new user
- Click on the settings button
- Open the "Security" section on the left an click on "Users" and then on the button "Create local user".
- Fill in the form (set 'nx-user' for the name and 'nx-user-pwd' for the password). Choose the "nx-anonymous" role for the moment.

**Step 3:** Create role
- Click on "Roles" on the left side and then on the button "Create role" in the top right corner.
- Choose the type 'Nexus role'
- Enter an ID and a name.
- Add the privilege "nx-repository-view-maven2-*-*" to the role and save it.

**Step 4:** Assign role to user
- Go back to the users and open the just created user. 
- Assign it the new role and remove the role nx-anonymous.

#### Steps to build a Jar and upload it to Nexus using Gradle
**Step 1:** Modify `build.gradle` file\
Make sure the `build.gradle` file of the [java-gradle-app](./sample-apps/java-gradle-app/build.gradle) contains the following elements:

```groovy
plugins {
    id 'maven-publish'
}

version '1.0.0-SNAPSHOT'

publishing {
    publications { // what do we want to publish
        mavenJava(MavenPublication) {
            artifact("build/libs/java-gradle-app-$version" + ".jar") {
                extension 'jar'
            }
        }
    }
    repositories { // where do we want to publish to
        maven {
            name 'nexus'
            def releasesRepoUrl = 'http://<nexus-ip-address>:8081/repository/maven-releases/'
            def snapshotsRepoUrl = 'http://<nexus-ip-address>:8081/repository/maven-snapshots/'
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

**Step 2:** Add `gradle.properties`\
Add a file called `gradle.properties` with the required user credentials to the project.
```sh
repoUser=nx-user
repoPassword=nx-user-pwd
```

**Step 3:** Build and publish
To publish the artifact to the Nexus repository (after having built it using `gradlew build`), just execute the command `gradlew publish`.

#### Steps to build a Jar and upload it to Nexus using Maven
**Step 1:** Modify `pom.xml` file\
Make sure the `pom.xml` file of the [java-maven-app](./sample-apps/java-maven-app/pom.xml) contains the following elements:

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
        <url>http://[nexus-ip-address]:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://[nexus-ip-address]:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

**Step 2:** Add credentials to `settings.xml`\
The credentials are defined in the file `~/.m2/settings.xml`:

```xml
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>nx-user</username>
            <password>nx-user-pwd</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>nx-user</username>
            <password>nx-user-pwd</password>
        </server>
    </servers>
</settings>
```

**Step 3:** Build and publish
To publish the artifact to the Nexus repository (after having built it using `mvn package`), just execute the command `mvn deploy`. Setting the version to a SNAPSHOT version or to a release version will be enough to make maven deploy the artifact to the snapshot repository or to the release repository respectively.