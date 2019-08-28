
#### Setup Kubernetes (K8s) Cluster on AWS


1. Create Amazon Linux EC2 instance - While creating key pair, download the key pair and save in your local machine. This will be required to connect your EC2 instance using Putty

1. Connecting to EC2 instance from Windows mahchine using Putty: Generate the private key using PuttyGen tool and connect. For more details follow: 
   https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html?icmpid=docs_ec2_console 
   ```sh
   Note: use default user name: ec2-user, for example
   a) hostname: ec2-ser@18.223.185.43 
   b) Navigate Conection -> SSH -> Auth from left side pane of putty and select the private key file which was generated using PuttyGen tool
   ```
   
1. install AWSCLI
   ```sh 
    sudo su -
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    yum install unzip python
    unzip awscli-bundle.zip
    #yum install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```
    
1. Install kubectl
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access
   ```sh
   AWSServiceRoleForElasticLoadBalancing
   AmazonEC2FullAccess
   IAMFullAccess
   AmazonS3FullAccess
   AmazonRoute53FullAccess
   ```
1. Attach IAM role to EC2 Linux server

    #### Note: If you create IAM user with programmatic access then provide Access keys. 
   ```sh 
     aws configure
    ```
1. Install kops on EC2 Linux instance:
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
1. create an S3 bucket 
   ```sh
    aws s3 mb s3://dev.k8s.kamalblog.in
   ```
1. Expose environment variable:
   ```sh 
    export KOPS_STATE_STORE=s3://dev.k8s.kamalblog.in
   ```
1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```
1. Create kubernetes cluster definitions on S3 bucket 
   ```sh 
    kops create cluster --cloud=aws --zones=us-east-2b --name=dev.k8s.kamalblog.in --dns-zone=kamalblog.in --dns private
    ```
1. Create kubernetes cluser
    ```sh 
      kops update cluster dev.k8s.kamalblog.in --yes
     ```
1. Validate your cluster 
     ```sh 
      kops validate cluster
    ```

1. To list nodes
   ```sh 
     kubectl get nodes 
   ```

#### Deploying Nginx container on Kubernetes 
1. Deploying Nginx Container
    ```sh 
      kubectl run sample-nginx --image=nginx --replicas=2 --port=80
      kubectl get pods
      kubectl get deployments
   ```
   
1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
   ```sh 
    kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
    kubectl get services -o wide
    ```
 1. To delete cluster
    ```sh
     kops delete cluster dev.k8s.kamalblog.in --yes
    ```
