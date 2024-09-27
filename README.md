## **Project: Deploy a Node.js Hello World App on Amazon EKS**

### **Objective**:
Deploy a simple Node.js web application on Amazon EKS, expose it to the internet, and scale it using Kubernetes.

---

### **Step 1: Build the Node.js "Hello World" Application**

#### 1.1 **Create a Simple Node.js App**

1. **Set up the Node.js project**:
   - Create a new directory for your project and navigate into it:
     ```bash
     mkdir hello-world-node
     cd hello-world-node
     ```

   - Initialize the Node.js project:
     ```bash
     npm init -y
     ```

2. **Install Express.js**:
   - Install the Express.js package, which will handle HTTP requests:
     ```bash
     npm install express
     ```

3. **Create `app.js`**:
   - Create a new file called `app.js` and add the following code to create a basic "Hello World" application:
     ```javascript
     const express = require('express');
     const app = express();
     const port = 3000;

     app.get('/', (req, res) => {
       res.send('Hello World from EKS!');
     });

     app.listen(port, () => {
       console.log(`App running on http://localhost:${port}`);
     });
     ```

4. **Test the Node.js app locally**:
   - Run the app locally by typing the following command:
     ```bash
     node app.js
     ```
   - Open your browser and visit `http://localhost:3000`. You should see the message: **"Hello World from EKS!"**

---

### **Step 2: Dockerize the Node.js Application**

Now that the Node.js app is running locally, let’s containerize it using Docker.

#### 2.1 **Install Docker**:
   - If Docker is not installed on your machine, follow the instructions [here](https://docs.docker.com/get-docker/) to install Docker for your platform (Windows, Mac, or Linux).

#### 2.2 **Create a Dockerfile**:
   - In your `hello-world-node` directory, create a file named `Dockerfile` with the following content:
     ```Dockerfile
     # Use the official Node.js image as the base image
     FROM node:14

     # Set the working directory inside the container
     WORKDIR /app

     # Copy the package.json and install dependencies
     COPY package*.json ./
     RUN npm install

     # Copy the rest of the application code
     COPY . .

     # Expose the app on port 3000
     EXPOSE 3000

     # Command to run the app
     CMD ["node", "app.js"]
     ```

#### 2.3 **Build the Docker Image**:
   - In the terminal, navigate to the directory containing the `Dockerfile` and build the Docker image:
     ```bash
     docker build -t hello-world-node .
     ```

#### 2.4 **Run the Docker Container Locally**:
   - After building the Docker image, run the container to test it:
     ```bash
     docker run -p 3000:3000 hello-world-node
     ```

   - Open `http://localhost:3000` in your browser. You should see the "Hello World from EKS!" message again, now running inside a Docker container.

#### 2.5 **Push the Image to Docker Hub or Amazon ECR**:
   - If you don’t have a Docker Hub account, create one [here](https://hub.docker.com/).
   - Log in to Docker from your terminal:
     ```bash
     docker login
     ```

   - Tag the Docker image:
     ```bash
     docker tag hello-world-node:latest <your-dockerhub-username>/hello-world-node:latest
     ```

   - Push the image to Docker Hub:
     ```bash
     docker push <your-dockerhub-username>/hello-world-node:latest
     ```

---

### **Step 3: Set Up Amazon EKS (Elastic Kubernetes Service)**

#### 3.1 **Install AWS CLI**:
   - Download and install AWS CLI from [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
   - After installing, configure AWS CLI:
     ```bash
     aws configure
     ```
   - Enter your AWS Access Key, Secret Key, and default region (e.g., `us-east-1`).

#### 3.2 **Install eksctl**:
   - **eksctl** is a CLI tool to create and manage EKS clusters. Install eksctl by following the guide [here](https://eksctl.io/introduction/#installation).
   
   - On Mac:
     ```bash
     brew tap weaveworks/tap
     brew install weaveworks/tap/eksctl
     ```

   - On Linux:
     ```bash
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
     sudo mv /tmp/eksctl /usr/local/bin
     ```

#### 3.3 **Install kubectl**:
   - **kubectl** is used to interact with your Kubernetes cluster. Install it from [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

   - On Mac:
     ```bash
     brew install kubectl
     ```

   - On Linux:
     ```bash
     sudo curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
     sudo chmod +x ./kubectl
     sudo mv ./kubectl /usr/local/bin/kubectl
     ```

---

### **Step 4: Create and Configure the EKS Cluster**

#### 4.1 **Create the EKS Cluster Using eksctl**:
   - Run the following command to create an EKS cluster (this will take about 15-20 minutes):
     ```bash
     eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 2 --nodes-min 1 --nodes-max 3 --managed
     ```

#### 4.2 **Configure kubectl to Access the EKS Cluster**:
   - After the cluster is created, run the following command to configure `kubectl`:
     ```bash
     aws eks --region us-east-1 update-kubeconfig --name my-eks-cluster
     ```

   - Verify the connection:
     ```bash
     kubectl get nodes
     ```

---

### **Step 5: Deploy the Node.js App to EKS**

#### 5.1 **Create Kubernetes Deployment YAML**:
   - In the same project directory, create a `deployment.yaml` file with the following content:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: hello-world-node
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: hello-world-node
       template:
         metadata:
           labels:
             app: hello-world-node
         spec:
           containers:
           - name: hello-world-node
             image: <your-dockerhub-username>/hello-world-node:latest
             ports:
             - containerPort: 3000
     ```

   - Replace `<your-dockerhub-username>` with your Docker Hub username.

#### 5.2 **Deploy the Application**:
   - Apply the deployment configuration to deploy your app to the EKS cluster:
     ```bash
     kubectl apply -f deployment.yaml
     ```

#### 5.3 **Expose the Application with a LoadBalancer**:
   - Create a `service.yaml` file to expose the app to the internet:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: hello-world-node-service
     spec:
       type: LoadBalancer
       selector:
         app: hello-world-node
       ports:
         - protocol: TCP
           port: 80
           targetPort: 3000
     ```

   - Apply the service configuration:
     ```bash
     kubectl apply -f service.yaml
     ```

   - Get the external IP address of your service:
     ```bash
     kubectl get service hello-world-node-service
     ```

   - Visit the external IP in your browser to see your Node.js app running.

---

### **Step 6: Scale the Application**

To scale the app up or down, modify the `replicas` in `deployment.yaml`:

```yaml
spec:
  replicas: 5  # Scale to 5 replicas
```

Apply the updated deployment:

```bash
kubectl apply -f deployment.yaml
```

You can check the status by running:

```bash
kubectl get pods
```
