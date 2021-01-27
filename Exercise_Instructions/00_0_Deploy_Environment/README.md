# Deploy workshop infrastructure and toolset

## Install JQ

```sh
$ sudo apt-get update -y
$ sudo apt-get install -y jq
```

## Deploy the EKS cluster and Istio

Install AWS CLI

```sh
$ sudo apt install awscli -yaws
```

Configure the AWS CLI
```sh
$ aws configure
```

Install kubectl
```sh
$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.12/2020-11-02/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
$ kubectl version --client
```

Install eksctl
```sh
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
$ eksctl version
```


With AWS CLI properly configured and kubectl and eksctl installed, run the following command to deploy the EKS cluster.

```sh
$ eksctl create cluster -f eks-cluster-ca.yaml
```

where eks-cluster-ca.yaml is:

```sh
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksHOT-test3
  region: ca-central-1

nodeGroups:
  - name: ng-1
    instanceType: t3.large
    desiredCapacity: 3
    volumeSize: 16
```
Then go to the istio directory:

$ cd dynatrace-k8s/istio
$ ./istio-install.sh


## Clone github repo

First download scripts and manifests required for this class from the github repo:
```sh
$ git clone https://github.com/steve-caron-dynatrace/dynatrace-k8s.git
```
Change directory to `dynatrace-k8s`. You can take a look at the deployment script:
```sh
$ cd dynatrace-k8s/dynatrace
```

## Gather environment and token info

To configure and deploy the OneAgent Operator, we will need the following info from your SaaS tenant:

- Environment ID
- API token
- PaaS token

Note: The installation procedure is also documented [here](https://www.dynatrace.com/support/help/shortlink/kubernetes-deploy) 

From your bastion host terminal, execute the following script to enter this info to have it stored in the `dynatrace.conf` file and environment variables so you don't need to type or copy-paste it later on.

```
$ source get-dt-cfg.sh
```
Again, you can always, if needed, get those configs printed by running this command (from the current directory):

```sh
$ cat dynatrace.conf
```

### Environment ID

For Dynatrace SaaS, the environment ID is your tenant ID. You can find it in the first part of your URL, e.g. `https://ENVIRONMENTID.sprint.dynatracelabs.com` .

- For example, for https://jwx05250.sprint.dynatracelabs.com , ENVIRONMENT ID=jwx05250

### API Token

Go in the left menu: <i>Settings -> Integration -> Dynatrace API</i>

1. Click on <b>Generate Token</b>

    ![generate_api_token](assets/generate_api_token.png)

2. Enter a name for your token (e.g. k8sOperator)
3. Leave the default options and click <b>Generate</b>
   
    ![api_token](assets/api_token.png)     

4. Expand the newly created token, copy the token value and paste it to your bastion terminal script prompt : API token 

    ![api_token_value](assets/api_token_value.png)


### PaaS Token

Go in <i>Settings -> Integration -> Platform as a Service</i>
1. Either copy the existing InstallerDownload token or click on <b>Generate Token</b>
   
    ![generate_paas_token](assets/generate_paas_token.png)

2. Enter a name for your token (e.g. k8sOperatorPaaS)
3. click <b>Generate</b>
   
    ![paas_token](assets/paas_token.png)

4. Expand the newly created token, copy the token value and paste it to your bastion terminal script prompt : PaaS token

    ![paas_token_value](assets/paas_token_value.png)


### Config Token

This is an additional token you will create. It is not needed for the Operator itself but it will be needed to automate some configurations in Dynatrace. This will, for example, create Web Application monitoring configuration and create Synthetic Browser Monitors to generate traffic to the Sock Shop web site.

- Follow the same procedure as for the <b><u>API token</u></b> except you will need to grant different access scope to this token than the default.

- Toggle on the following options:

  - Create and read synthetic monitors, locations, and nodes
  - Read configuration
  - Write configuration

    ![config_token](assets/config_token.png)

- Don't forget to click the <b>Generate</b> button!
- Expand the newly created token, copy the token value and paste it to your bastion terminal script prompt : Config token

---

[Previous : #1 Deploy the Sock Shop app](../01_Deploy_Sock_Shop) :arrow_backward: :arrow_forward: [Next : #3 Automatic import of k8s labels and annotations](../03_Import_k8s_labels_annotations)

:arrow_up_small: [Back to overview](../)