# Deploy Full Stack Application using CICD Pipeline

**Deploy Full Stack Application using Kubernetes, Docker, Jenkins, SonarQube, Maven, Nexus**

## WORKFLOW OF THE APPLICATION | ARCHITECTURE 

**🌟 Key Moments 🌟**

- Configuring POM.xml
- Setting up "Managed Files" => For Global Maven Settings
- Setting Up Pipelines
- Configuring the Control Plane and Data Plane Nodes in Kubernetes
- Deploying to Kubernetes

Prerequisites:
Basic knowledge of Kubernetes, Jenkins, and AWS Account

[MUST] Clone the Repository [For getting the Application Code]

[GitHub Repository](https://github.com/naveensilver/Ekart-Project.git)

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
```
# Default Credentials: 

Name: admin
Password: cat <Path given in dialog box> (i.e., `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`)
```
Explanation:
- These are the default credentials provided during Jenkins installation.
- To retrieve the password, you can use the command `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` and paste the password in the Jenkins login prompt.

#### Setting up New Credentials

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

### Configure Tools [Jenkins Dashboard]

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


# 4. Setting Pipelines [Jenkins Dashboard]

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

# 5. Setting Up the Kubernetes Cluster

To create a Kubernetes cluster on AWS, we need to set up 3 nodes with the following configuration:

- 1 Master Node
- 2 Worker Nodes

Each node should have the following basic configuration:
- Instance type: t2.medium
- Storage: 15 GB
- Instance count: 3

Additionally, we need to ensure that the same Security Group used for Jenkins, Nexus, and Sonar is applied to these nodes.

Furthermore, we should rename the nodes as follows:
- Master Node: k8-Master
- Worker Nodes: Worker-01, Worker-02

### Setting Up the MasterNode and Worker Nodes [Using Kubeadm]

To set up the Master Node and Worker Nodes using Kubeadm, follow these steps:

1. Update the package repository and install Docker:

```bash
sudo apt-get update -y
sudo apt-get install docker.io -y
sudo service docker restart
```
2. Add the Kubernetes repository and update the package repository:

```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >/etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```
3. Install Kubernetes components:

```
sudo apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
```
Run all the above commands in both the Master and Worker Nodes. These commands will install Docker and Kubernetes components required for setting up the Master Node and Worker Nodes.

### Setting Up the Master Node ONLY

To set up the Master Node only, follow these steps:

Initialize the Kubernetes master node and specify the Pod Network CIDR:

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```
After running the above command as the root user, it will generate a link to join other worker nodes at the end like the following:

```
kubeadm join 172.31.89.0:6443 --token iotnjn.b50nhy71wg2be2pc \
    --discovery-token-ca-cert-hash sha256:d5cba19e1f3be8657c90e143a5b671ea9b39c9c5e749c6c66116a0a1ce3fecc9
```
Use this command to join other worker nodes to the cluster.


**NOTE:** You can add the link to any number of VMs, but they must have the required `kubeadm` and `kubectl` installations preconfigured to function as Worker Nodes.

Since we have designated Worker-01 and Worker-02 as dedicated worker nodes, we will use them for adding worker nodes. Run the above command on the worker nodes. After execution, it will display a message like this:
```
message: node has joined the cluster.
```

Additionally, the Master Node will provide output like the following:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf
```

We have two options for configuring the Kubernetes cluster:

1. Generate the "kubeconfig" file as a non-root user:
   - Run the following commands as a regular user:
     ```
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     ```

2. Use the root user and export the configuration:
   - Run the command:
     ```
     export KUBECONFIG=/etc/kubernetes/admin.conf
     ```

Once the configuration is set up, proceed to install the CNI Plugin (Calico) and Ingress Controller in the master plane:

```bash
# Install Calico (by manifest files)
kubectl apply -f https://docs.projectcalico.org/v3.20/manifests/calico.yaml

# Install Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```
### Creation of Service Account

**What is a Service Account?**

A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster. Application Pods, system components, and entities inside and outside the cluster can use a specific ServiceAccount's credentials to identify as that ServiceAccount.

Service accounts are a namespace.

Service accounts exist in the cluster and are defined in the Kubernetes API. You can quickly create service accounts to enable specific tasks.

We need to create the service account for the Jenkins user so that it can create the necessary Kubernetes resources when the application gets deployed.
```
# Add a YAML file for ServiceAccount
# Create sa.yml file with the following content:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps

# Here in our master node the namespace: `webapps` does not exist so Create the namespace 'webapps'
kubectl create namespace webapps

# Apply the changes to the YAML file
kubectl apply -f sa.yml
```
### Creation of Role [For webapps Namespace]

Roles in Kubernetes define the actions that a user can perform within a cluster or namespace. These roles can be assigned to Kubernetes subjects such as users, groups, or service accounts using role bindings and cluster role bindings.

For our specific scenario, we are creating a role within the `webapps` namespace to govern the creation of Kubernetes resources. This role will grant permissions for various actions like getting, listing, watching, creating, updating, patching, and deleting resources such as pods, deployments, services, and more.

To implement this, we need to define a role YAML file (`role.yml`) with the desired permissions, and then apply this YAML file to the Kubernetes cluster using the `kubectl apply` command. This will create the role named `app-role` within the `webapps` namespace.

```
# Create role.yml file with the following content:
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
       - ""
       - apps
       - autoscaling
       - batch
       - extensions
       - policy
       - rbac.authorization.k8s.io
    resources:
       - pods
       - componentstatuses
       - configmaps
       - daemonsets
       - deployments
       - events
       - endpoints
       - horizontalpodautoscalers
       - ingress
       - jobs
       - limitranges
       - namespaces
       - nodes
       - persistentvolumes
       - persistentvolumeclaims
       - resourcequotas
       - replicasets
       - replicationcontrollers
       - serviceaccounts
       - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

```

Apply the changes to create the role:

```
kubectl apply -f role.yml
```
This will create a role named `app-role` in the `webapps` namespace with the specified permissions.


### Creation of Role Binding:

To establish permissions for the Jenkins user within the `webapps` namespace, we need to create a role binding. This binding links the previously created role (app-role) with the Jenkins service account.

The role binding YAML file (assign.yml) defines the association between the role and the service account, granting the Jenkins user the permissions specified in the role.

Here's the content of the `assign.yml` file:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
- namespace: webapps
  kind: ServiceAccount
  name: jenkins
```
After applying this configuration, Jenkins will possess the necessary permissions to manage Kubernetes resources within the `webapps` namespace. This setup adheres to best practices by utilizing a service account instead of the root account, ensuring proper access control and security within the Kubernetes environment.

To apply the configuration defined in the `assign.yml` file and create the role binding, you can use the following command:

```bash
kubectl apply -f assign.yml
```

This command will instruct Kubernetes to read the configuration from the `assign.yml` file and apply it to the cluster. As a result, the role binding named `app-rolebinding` will be created in the `webapps` namespace, linking the `app-role` with the `jenkins` service account.

Now using this our Jenkins user will have all the necessary permissions to create the pods and other Kubernetes resources.

In real-time; we do not give the root account rather we could use the "service account" and add the "role" to manage the "role-binding" to give necessary permissions to Kubernetes Resources;

**Note:**In Kubernetes, it's best practice to use service accounts with appropriate roles and role bindings to grant permissions to applications or users, rather than using the root account. By using service accounts and role-based access control (RBAC), you can enforce the principle of least privilege, ensuring that each component or user has only the permissions necessary to perform its intended tasks. This enhances security and reduces the risk of unauthorized access or accidental damage to the cluster.

### Create Token for Service Account [Creation of Secret]

Creating a token (secret) for a service account is essential for authenticating with the Kubernetes API. It's a standard practice to have a secret for every service account to ensure secure communication between components within the Kubernetes cluster.

Here's how to create the secret.yml:

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-acount-token
metadata:
  name: mysecretname
  annotations: 
    kubernetes.io/service-account.name: jenkins
```

In the above secret configuration, we haven't specified any namespace. However, we need to add this secret to our newly created namespace, which is "webapps":

```bash
kubectl apply -f secret.yml -n webapps
```

This ensures that the Jenkins service account has a token (secret) available for authentication within the "webapps" namespace.

To view the secret token, you can use either of the following commands:

```bash
kubectl -n webapps describe secret mysecretname
```

or

```bash
kubectl describe secret jenkins-token-<hash> -n webapps
```

Here is the token:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IlBLRmdGN2NsdFJqbjhaYmxycEp4VndxdGFsTWhGbEdVdldvMUpMQlRvbjAifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImplbmtpbnMtdG9rZW4tZGhjcTYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiamVua2lucyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBkZTBjOWE3LWMyNDgtNDZlOC04OTgyLTJiYjNhMmM1NjQ0MSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDp3ZWJhcHBzOmplbmtpbnMifQ.BjlGDEIMPWGagi9cTB2a7HdG3IsrXjNy5_JxtE2UVyv_XQZwHP_AIEU021SQmrXcvMwe6F2sSmhidvKjuQgBp15i_1lQadaqca0p1KWXSXaD2lEjpdg2qlbc46Hjjn6kqNmyhAsD_w2xQdAHooZffWyUgypzWfl3JP6Vz1_cbPtrcOy6MmSwzTWAlf3zid4BqrdddAwIHxQJfaY2LmO0qPNrhry5nazDw2_XRnaRrhELbpZp6f_j-fnTxVimaVo2ODS6aCOgmyLt4rlOsRJajOxGb25FqRp2kr0VxCJM_WhTvZIgcrjmfqlrFtrjetv-AkbJZG1h5Es497YUGKUcMw
```

This token can be used for authentication and authorization purposes within the Kubernetes cluster.

**Find the kubeconfig file in the Master**

To locate the kubeconfig file in the Master node, follow these steps:

```bash
# Navigate to the directory where kubeconfig file is stored
cd ~/.kube

# View the contents of the config file
cat config
```

The `config` file contains all the necessary configurations of the Kubernetes cluster, including cluster information, authentication details, and context settings.



### Stage 8: Docker Build and Tag:

To configure the `Docker Registry Endpoint`, utilize the Pipeline Syntax:

**Docker RegistryURL:** Do not fill it (As it will contain only Public DockerHub Repository)
**Registry Credentails:** Add the Docker Credentials Specified in the Credentials

The block will look like:

```groovy
withDockerRegistry(credentialsId: 'dockerhub-credentials') {
    // some block
}
```

Now, to build the image, use the docker build command:

```groovy
sh "docker build -t <DOCKERHUB_NAME>/<REPOSITORY NAME>:<TAG NAME> -f docker/Dockerfile ."
sh "docker build -t adityadho/ekart:latest -f docker/Dockerfile ."
```

So our Docker Build and Tag Block looks as:

```groovy
stage('Docker Build and Tag') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'dockerhub-credentials') {
               sh "docker build -t adityadho/ekart:latest -f docker/Dockerfile ."
            }
        }
    }
}
```

### Stage 9: Trivy for Vulnerability Scans

Since Trivy is not installed yet, it needs to be installed in the Jenkins Server.

```
#Install Trivy:

sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

OR

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.deb
sudo dpkg -i trivy_0.18.3_Linux-64bit.deb
```

To scan the image using trivy:

```bash
trivy image <image-name>
sh "trivy image adityadho/ekart:latest"
```

But we will always need to log onto the console output to see the changes; rather than that we could store the report in some text file as below:

```bash
sh "trivy image adityadho/ekart:latest > trivy_report.txt"
```

Our Trivy Vulnerability scan will look like:

```groovy
stage('Trivy Scanning') {
    steps {
        sh "trivy image adityadho/ekart:latest > trivy_report.txt"
    }
}
```


### Stage 10: Pushing Docker Image to the DTR

This stage will consist of the same steps as that of the Docker Build and Tag stage.

We will use the docker push command to push the image into the Docker Trusted Repository.

```groovy
stage('PUSH Image to DTR') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'dockerhub-credentials') {
               sh "docker push adityadho/ekart:latest"
            }
        }
    }
}
```


### Stage 11: Deploy using Kubernetes:

In this stage, we will make use of the Pipeline Syntax `withKubeConfig` to configure Kubernetes.

Before that, add a global credential and then add the secret text. The secret text will be the token that we have created above.

```groovy
withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.89.0:6443') {
    // some block
}
```

Before adding the Deployment steps, we need to make some changes in the `deploymentservice.yml`.

Changes in the `deploymentservice.yml`:

```yaml
- Change the Image name to "image: adityadho/ekart:latest"
- Change the Deployment ContainerPort from "8080" to "8070" [As in the Dockerfile we have exposed the 8070 and our application is running on PORT 8070]
- Change the Port and targetPort both to 8070 [type: Nodeport]
- PORT => On which the service is running on the Cluster
- targetPort => Port on which our application is running
```


**Deploy the Kubernetes Cluster**

To deploy the Kubernetes cluster, run the following commands:

```sh
kubectl apply -f deploymentservice.yml -n webapps
kubectl get svc -n webapps
```

Ensure that Jenkins has the "kubectl installation" while running the pipelines. You can install it using the Jenkins Server:

```sh
sudo snap install kubectl --classic
```
Try and build the pipeline. The pipeline will run through all the stages.

### Results 

1. Jenkins Pipeline Dashboard
2. Docker image Dashboard
3. Our Sonar Dashboard
4. Nexus Repo Dashboard

```sh
# To check the service running:

kubectl get svc -n webapps

# To log into the application:

username: admin
password: admin

http://<worker-node-ip>:<generated-port-number-console-o/p>/home

Perform any operations; it should work on that application.
```
**Working Demo**

Video

## Full Pipeline:
```
```


**Conclusion:**

Throughout this process, we've successfully developed the entire CI/CD pipeline and created the Jenkins file from scratch. I trust you found this journey engaging and insightful. Feel free to experiment and explore further on your own. Happy Learning!

Source: https://youtu.be/yH9RoOFEKMY?si=j6qIRo3x3MBUG9gh 
