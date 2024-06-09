

## Setting up the Instances and Security Groups

Create individual machines for Jenkins, Nexus, and Sonarqube, each with the following specifications:

### EC2 Machine Details:
- **Instance Type:** t2.medium
- **Storage:** 20 GB
- **Operating System:** Ubuntu 20.04

For Security Groups, add the following groups:

1. **SMTP TCP 25:** This allows communication via SMTP on port 25.
2. **Custom TCP - TCP Range from 30000 to 32767:** Reserved for Kubernetes NodePort Service, enabling communication within the specified port range.
3. **Custom TCP - TCP Range from 3000 to 10000:** Reserved for maximum installations of tools within this port range.
4. **HTTP - TCP - 80:** Allows HTTP communication over port 80.
5. **HTTPS - TCP - 443:** Enables secure HTTP communication over port 443.
6. **SSH TCP 22:** Permits SSH communication over port 22 for remote access.
7. **CUSTOM TCP 6443:** Specifically designated for Kubernetes API server, listening on port 6443 on the first non-localhost network interface, protected by TLS.

These configurations ensure the secure operation and communication channels required for Jenkins, Nexus, and Sonarqube instances, alongside the necessary protocols and ports for their functionality and integration within your environment.

### Configuration Changes:
1. Set up Jenkins, Nexus, and SonarQube on the same instance.
2. Increase the instance count 1 to 3.

### Note:
Running Jenkins, Nexus, and SonarQube on the same instance may lead to resource contention and performance issues, especially if the instance type is not powerful enough to handle the workload. Consider upgrading the instance type or distributing the services across multiple instances if performance becomes a concern.

# 1. Jenkins Server Setup:

**Install Java and Jenkins:**
Run the following commands in your terminal or command prompt:

```
# Update package lists
sudo apt-get update

# Install Java 17
sudo apt install openjdk-17-jre-headless

# Add Jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package index
sudo apt-get update 

# Install Jenkins
sudo apt-get install jenkins

# Enable Jenkins service to start on boot
sudo systemctl enable jenkins

# Start Jenkins service
sudo systemctl start jenkins

# Check Jenkins service status
sudo systemctl status jenkins
```
**Install Docker in Jenkins VM"**
```
#installs Docker on your system
sudo apt install docker.io

# Adjust Docker Permissions: 
sudo chmod 666 /var/run/docker.sock
```
**Access Jenkins Dashboard:**

- After installation, access the Jenkins Dashboard from your web browser at `http://<IP_of_Jenkins_VM>`:8080.

**Jenkins Dashboard Login:**

### Default Credentials

- **Name:** admin
- **Password:** cat <Path given in dialog box> (i.e., `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`)

Explanation:

- These are the default credentials provided during Jenkins installation.
- To retrieve the password, you can use the command `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` and paste the password in the Jenkins login prompt.

### Setting up New Credentials

- **Username:** admin
- **Password:** admin@123

Explanation:
- After logging in with the default credentials, you can set up new credentials with username 'admin' and password 'admin@123' for future logins.

### Install Plugins in Jenkins [Jenkins Server]

1. Log in to your Jenkins dashboard.
2. Navigate to "Manage Jenkins" > "Manage Plugins" > "Available" tab.
3. Search for the following plugins:
   - SonarQube Scanner
   - Pipeline Maven Integration Plugin
   - Nexus Artifact Uploader
   - Config File Provider  
   - Docker
   - Docker Pipeline
   - Docker Build Step
   - CloudBees Docker Build and Publish
   - OWASP Dependency Check
   - Eclipse Temurin Installer # For JDK - helps to install multiple jdk version
   - Kubernetes
   - Kubernetes CLI
4. Check the checkboxes next to the plugins you want to install.
5. Click on the "Install without restart" button to install the selected plugins.

### Configure Tools [JENKINS SERVER]

1. Navigate to "Manage Jenkins" > "Tool Configuration".

2. Configure JDK
- Install JDK from Adoptium with a version above jdk17.3.4.
  - Set the name as "jdk17".
- If the application doesn't work with JDK17, install JDK11 as well.
  - Install JDK from Adoptium with a version above jdk11.3.4.
  - Set the name as "jdk11".

3. Sonar Scanner Installations
- Install the Sonar Scanner with the latest version.
  - Set the name as "sonar-scanner".

4. Maven Installations
- Install Maven with a version above Maven 3.6.3.
  - Set the name as "maven3".

5. Dependency Check Installation
- Install the OWASP Dependency Check plugin automatically with version 6.5.1.
  - Set the name as "DC".

6. Docker Installations
- Install Docker from docker.com automatically with the latest version.
  - Set the name as "docker".

# 2. SonarQube Server Setup:

**Configure the SonarQube Server using Docker:**

First, you need to install Docker on your system. Follow these steps:
```
# updates the package index to ensure you get the latest version of Docker
sudo apt update

#installs Docker on your system
sudo apt install docker.io

# Adjust Docker Permissions: 
sudo chmod 666 /var/run/docker.sock

#The `/var/run/docker.sock` file is used by Docker to communicate with the Docker daemon. Changing permissions to `666` allows all users to interact with Docker. However, this is not recommended for security reasons.
```
**Run SonarQube Docker Container:**
```
docker run -d -p 9000:9000 sonarqube:lts-community
```
- The `docker run` command starts a new Docker container.
- The `-d` flag runs the container in detached mode, meaning it runs in the background.
- The `-p 9000:9000` flag maps port 9000 of the container to port 9000 of the host machine, allowing access to SonarQube's web interface.
- `sonarqube:lts-community` is the Docker image for the SonarQube LTS(Long Term Support) community edition.

By following these steps, you'll have SonarQube Server up and running as a Docker container on your system.

**Sonar Dashboard:**

- After installation, access the Jenkins Dashboard from your web browser at `http://<Public_IP_of_SonarQube_VM>`:9000

**Sonar Dashboard Login:**

Default Credentials

- **Username:** admin
- **Password:** admin

Note: Update new password

### Connecting SonarQube to Jenkins

1. **Create Token in SonarQube:**
   - Log in to SonarQube Dashboard.
   - Go to Administration > Security > Users > 
   - Generate a new token with Jenkins integration permissions for Jenkins intigration.
      - Name: sonar-token   (i.e., squ_<hash_values>)
   - Take note of the generated token as it will be required in the next step.

2. **Add the Token into Jenkins Global Credentials:**
   - Log in to your Jenkins dashboard.
   - Go to "Manage Jenkins" > "Credentials".
   - Click on "Global credentials (unrestricted)" or any other appropriate scope.
   - Click on "Add Credentials".
   - Select "Secret text" as the kind of credentials.
   - Enter the following details:
     - **Secret:** [Token generated from SonarQube, e.g., squ_<hash_values>]
     - **ID:** sonar-token
     - **Description:** sonar-token
   - Click on "OK" or "Save" to add the credentials.

3. **Configure the Sonar server:**
  - Go to Jenkins Dashboard > Manage Jenkins > System Configuration - System (Here we can configure servers)
  - Go to SonarQube servers > SonarQube installations > Add SonarQube
    - Name: sonar
    - Server URL: http://<Sonar_Server_IP>:9000
    - server authentication token: sonar-token   #token


By following these steps, you'll successfully create a token in SonarQube and add it to Jenkins, enabling integration between the two platforms for code analysis and quality management.

**Configure Jenkins to Dockerhub:**

- Add the Dockerhub credentials to the Global Credentials section.
  - Enter your Dockerhub username and password (or Generate token from dockerhub) as username with password type.
    - Username: <your_dockerhub_username>
    - Password: <your_dockerhub_password_or_token>
    - ID: docker-cred
    - Description: docker-cred

# 3. Nexus Server Setup:

**Configure the Nexus Server using Docker:**

First, you need to install Docker on your system. Follow these steps:
```
# updates the package index to ensure you get the latest version of Docker
sudo apt update

#installs Docker on your system
sudo apt install docker.io

# Adjust Docker Permissions: 
sudo chmod 666 /var/run/docker.sock
```
**Run Sonar Nexus as Docker Container:**
```
# Install Sonar Nexus using docker image 
docker run -d -p 8081:8081 sonatype/nexus3

# To Check the running docker container 
docker ps
# To login to the NEXUS DB we need username and password;
# To get the password. 
docker exec -it <ID_of_NEXUS_CONTAINER> /bin/bash

# the password is stored at the location
cd sonatype-work/nexus3
cat admin.password
```
Now, exit from the container using "exit"

**Access the Nexus Dashboard**
```
- Go to Browser and Run: <Public_IP_Nexus_VM>:8081

# enter the credentials
username: admin
password: it is stored in the sonatype-work/nexus3 [as seens above]

# set new password as per your liking as we will need it further
new username: admin
new password: your new password
```
We are going to use the "maven-release" and "maven-snapshot" repositories; when pushing the repos we generally push our builds in the "maven-snapshot" but only when we want to release it to the production we have set the repository to the "maven-release". These repositories are set in the POM Files; POM files contain the dependency of the projects;

**Connect Jenkins to Nexus:**

1. **Access Nexus Dashboard**

    - Open your browser and navigate to `http://<IP of NEXUS Server>:8081` and log in with the admin credentials.

2. **Configure Managed Files in Jenkins**
Here, we install `Config File Provider` Plugin, then we get `Managed Files` Option.
    - Navigate to: `Manage Jenkins -> Managed Files -> Add a new config file`
    - **Add new configuration**:
        - Type: `global maven settings.xml`
        - ID: `global-maven`
        - Content: `<credentials of the nexus server>`

In the content section you will find below block from under the commented section and add them into config file.
```
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
```
Alter the contents of them like "maven-release" and "maven-snapshot"

3. **Configure Nexus Credentials in Config File Management**

    - **For Maven Release**:
        ```
        <server>
          <id>maven-releases</id>
          <username><YOUR USERNAME for NEXUS></username>   #add nexus username
          <password><YOUR PASSWORD for NEXUS></password>   #add nexus password 
        </server>
        ```

    - **For Maven Snapshot**:

        ```
        <server>
          <id>maven-snapshots</id>
          <username><YOUR USERNAME for NEXUS></username>   #add nexus username
          <password><YOUR PASSWORD for NEXUS></password>   #add nexus password
        </server>
        ```
Now, Click on submit.

4. **Add Repositories in POM.xml**

Go to GitHub > POM.XML and Add the URL for both the `maven-release` and `maven-snapshot` in `<distributionManagement> </distributionManagement>` tag
    ```
    <repository>
      <id>maven-releases</id>
      <name>maven-releases</name>
      <url><URL LINK FOR THE MAVEN RELEASE FROM NEXUS REPOSITORIES></url>
    # <url>http://<Nexus_Public_IP>:8081/repository/maven-releases/</url>

    </repository>

    <snapshotRepository>
      <id>maven-snapshots</id>
      <name>maven-snapshots</name>
      <url><URL LINK FOR THE MAVEN SNAPSHOTS FROM NEXUS REPOSITORIES></url>
    # <url>http://<Nexus_Public_IP>:8081/repository/maven-snapshots/</url>

    </snapshotRepository>
    ```
### Using Nexus Repositories 

We will be using two repositories in Nexus: `maven-release` and `maven-snapshot`.

- **maven-release Repository**: Store stable, production-ready artifacts.
  - It means that the artifact is ready for release; we will store it under the maven-release repository.
  - Example Version: `0.0.1`

- **maven-snapshot Repository**: Store development or in-progress artifacts.
  - It means that the generated artifact is still in the development phases and it is not yet ready for production(release) we will store it under the maven-snapshot repository
  - Example Version: `0.0.1-SNAPSHOT`

Where, we will push our artifact that the Help of the POM.xml file will decide under the <versions></versions> it can either be 0.0.1-SNAPSHOT or 0.0.1

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

#4. Setting Pipelines [Jenkins Dashboard]

### Foundation of the Pipeline

1. **Create a New Pipeline Job:**
    - Open Jenkins Dashboard.
    - Click on `New Item`.
    - Enter an item name (e.g., `full-stack`).
    - Select `Pipeline` and click `OK`.

2. **Configure the Pipeline Job:**
    - **General Settings**:
      - Optionally, add a description of the pipeline.
    - **Discard Old Builds**:
      - Check the `Discard Old Builds` option.
      - Enter the following:
        ```sh
        No. of Builds to Keep: 2
        Days to keep Build: 100 Days
        ```
    - **Pipeline Definition**:
      - Scroll down to the `Pipeline` section.
      - Choose `Pipeline script` from the `Definition` dropdown.
      - In the `Script` text box, enter your pipeline code.

Note: No two pipeline stage names should be the same.

## Jenkins Pipeline Stage Creation

### Stage 1: Checkout

The Checkout stage creates a local copy of the source code in Jenkins in the Workspace.

#### Generating the Syntax for the Pipeline

To generate the syntax for the pipeline, you can use the "Pipeline Syntax" generator in Jenkins. The syntax for checking out code from a Git repository would look like this:

```
git branch: 'main', url: 'https://github.com/adityadhopade/Ekart.git'
```
Here is how Stage 1 will look in the Jenkins pipeline script:
```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adityadhopade/Ekart.git'
            }
        }
    }
}
```

### Stage 2: Compile Source Code

As our base project is a SpringBoot Application, we have used Maven as the Build Tool. Therefore, we will use Maven to compile our code.

The command to compile the source code using Maven is simply `mvn compile`. However, we should enclose it within the shell commands in the Groovy pipelines.

Also, as we are using Maven as a tool here, we can define it in the `tools {}` block to ensure its version consistency across the pipeline. Use it below:

```
tools {
    maven 'maven3'
    jdk 'jdk17'
}
```
So, the stage 2 should look like this:
```
pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adityadhopade/Ekart.git'
            }
        }

        stage('Compile Code') {
            steps {
                sh "mvn compile"
            }
        }
    }
}
```
### Stage 3: Unit Test

Run the unit test cases using Maven. Although the test cases fail in this project, they are still useful in the development phase. However, failing test cases should not affect the production environment.

The command to run the test cases is `mvn test`, but we need to skip the test cases for now. This can be done with the help of the `-DskipTests=true` flag.

```groovy
sh "mvn test -DskipTests=true"
```

Our Stage 3 will look like this:
```
pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adityadhopade/Ekart.git'
            }
        }

        stage('Compile Code') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
    }
}
```
### Stage 4: SonarScan Analysis

Sonar Scanner is utilized to obtain information about Code Smells, Bugs, Quality Gates, and Quality Profile of our applications.

#### Setting Environment Variables for Sonar Scanner

We can set the environment variable for the Sonar Scanner, SCANNER_HOME, using the `environment` directive:

```groovy
environment {
    SCANNER_HOME = tool 'sonar-scanner'
}
```

Make use of the Pipeline Syntax Generator: To use the Sonar Scanner, we can utilize the withSonarQubeEnv block. We'll add the generated token (sonar-token) to access the Sonar dashboard.
```
withSonarQubeEnv(credentialsId: 'sonar-token', installationName: '') {
    // some block
}
```
Instead of providing the credentialsId of the server, we can simply pass the Sonar server that we have configured:

```
withSonarQubeEnv('sonar') {
    // Sonar Scanner commands will go here
}
```
To scan the project, we can utilize the executable stored under `$SCANNER_HOME/bin/sonar-scanner`. When scanning the project, we need to provide the following details:

- Pass the name of the project: `-D sonar.projectKey=EKART`
- Pass the project name: `-D sonar.projectName=EKART`
- Pass the Java binaries (as our project is Java-based and consists of binary files): `-D sonar.java.binaries=.`

Now the Block will look like:
```
withSonarQubeEnv('sonar') {
    sh ''' $SCANNER_HOME/bin/sonar-scanner -D sonar.projectKey=EKART -D sonar.projectName=EKART -D sonar.java.binaries=.
}
```

Our Stage 4 will look like this:

```
pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adityadhopade/Ekart.git'
            }
        }

        stage('Compile Code') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -D sonar.projectKey=EKART -D sonar.projectName=EKART \
                    -D sonar.java.binaries=. '''
                }
            }
        }
    }
}
```
### Stage 5: OWASP Dependency Check

OWASP Dependency Check is utilized to detect and report publicly disclosed vulnerabilities, thereby enhancing application security. The generated report can be saved under our project for reference.

To integrate OWASP Dependency Check into our pipeline, we can use the Pipeline Syntax Generator > `DependecyCheck: Invoke Dependency` Check as follows:

```
dependencyCheck additionalArguments: '', odcInstallation: 'DC'
```
We also need to provide additional arguments to scan the dependency files present under the "Pom.xml". We can accomplish this by using the `--scan` flag. Since the `Pom.xml` is located in the root directory, we specify it as `./`.

```groovy
dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
```

OWASP Dependency Check generates the report in XML format. To specify the desired format for the report, we use the Dependency Check Format Generator.

```groovy
**/dependency-check-report.xml
```

Our pipeline block for specifying the report format will look like this:
```
dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
```
Our pipeline block Stage 5 for OWASP Dependency Check will look like this:

```
stage('OWASP Dependency Check') {
    steps {
        dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
```
### Stage 6: Build Artifact using Maven

To build the application artifact using Maven, we execute the command `mvn package`. Since we have skipped the test cases in an earlier stage, we need to include the `-DskipTests=true` flag to skip the tests during the build process.

```groovy
sh "mvn package -DskipTests=true"
```
The stage 6 for building the artifact using Maven will look like this:

```
stage('Build Application') {
    steps {
        sh "mvn package -DskipTests=true"
    }
}
```

### Stage 7: Deploy to Nexus Repository

In the `pom.xml` file, the artifact is configured as a snapshot, which means it will be pushed into the "maven-snapshot" repository in the development stage, as it's not yet at the production level.

#### Using Pipeline Syntax with Maven Environment

We'll make use of the `withMaven` directive to provide Maven environment settings. This allows us to set the Maven version, JDK version, and utilize the global Maven settings configuration that we have set during the configuration of "Connect Jenkins to Nexus" [JENKINS SERVER].

```groovy
withMaven(globalMavenSettingsConfig: 'MyGlobalSettings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
    // Deploy to Nexus repository steps will go here
}
```

To deploy our artifact to the Nexus Repository, we can utilize the Maven command `mvn deploy`. 

So, the stage 7 deploy artifact to nexus repository looks like :

```groovy
stage('Deploy to Nexus') {
    steps {
        withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
            sh "mvn deploy -DskipTests=true"
        }
    }
}
```
