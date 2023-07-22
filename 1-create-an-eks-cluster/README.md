# Step 1: Create an EKS cluster

The following steps guide you through the process of creating an EKS cluster on AWS, with the help of Pulumi IaC
framework. It takes about **_30_** minutes to complete.

If you have already installed the AWS CLI and Pulumi, you can skip the corresponding steps

1. Set up AWS CLI

    1. Install AWS CLI

        - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
        - `[aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)`:
          Amazon EKS uses IAM to provide secure authentication to your Kubernetes cluster.

    2. Config AWS credentials

       Run this command to quickly set and view your credentials, region, and output format. The following example shows
       sample values.

       ```bash
       $ aws configure
       AWS Access Key ID [None]: XXXXXXXXX
       AWS Secret Access Key [None]: XXXXXXXXX
       Default region name [None]: ap-southeast-1
       Default output format [None]: yaml
       ```

2. Set up Pulumi

    1. Install Pulumi

       https://www.pulumi.com/docs/get-started/install/

    2. Login to Pulumi

       ```bash
       $ pulumi login --local
       ```

    3. Set passphrase env to `""`

       > This passphrase is required by Pulumi and was created by Lab maintainer.

       ```bash
       $ export PULUMI_CONFIG_PASSPHRASE=""
       ```

    4. Select the `default` stack
    
       ```bash
       $ cd 1-create-an-eks-cluster
       # remove the `default` stack created yesterday to prevent conflicts

       $ pulumi stack rm default -f
       
       $ pulumi stack select default -c
       ```

3. Create the EKS cluster via Pulumi (may take more than **_30_** minutes)

    ```bash
    $ pulumi up
    Updating (default):

         Type                      Name                            Status
    +   pulumi:pulumi:Stack        1-create-an-eks-cluster-default created
    +   └─ eks:index:Cluster       my-eks                          created
        ... dozens of resources omitted ...
     
    Do you want to perform this update?  [Use arrows to move, type to filter] 
    > yes
      no
    ```

4. Interact with the newly created EKS cluster

    ```bash
    $ pulumi stack output kubeconfig > kubeconfig.yaml
    $ export KUBECONFIG=$PWD/kubeconfig.yaml

    $ kubectl get nodes
    NAME                                            STATUS   ROLES    AGE   VERSION
    ip-xxx-xxx-xxx-xxx.ap-southeast-1.compute.internal   Ready    <none>   2m   v1.27.1-eks-2f008fe
    ip-xxx-xxx-xxx-xxx.ap-southeast-1.compute.internal   Ready    <none>   2m   v1.27.1-eks-2f008fe
    ```