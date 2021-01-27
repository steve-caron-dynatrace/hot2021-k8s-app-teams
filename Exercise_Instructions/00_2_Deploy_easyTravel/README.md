# Exercise #1 Deploy the easyTravel app

## Script overview

Change directory to `dynatrace-k8s/easytravel/scripts`. You can take a look at the deployment script:
```sh
$ cd dynatrace-k8s/easytravel/scripts
$ cat create_easytravel.sh
```
The script does the following tasks:
- Deploy a MongoDB datastore service
- Deploy the Tomcat-based backend and frontend application services
- Deploy an nginx http reverse proxy (www service)
- Deploy a load generator and a plugin services
- Expose the wwww service to the public internet

## Deploy Sock Shop
Execute the deployment script:
```sh
$ ./create_easytravel.sh
```
## Validate
Check the command output and validate the pods deployed.

Notice the pods status and ready state. They should all be `Running` and `Ready`. 

You can always get the list of easyTravel pods and services by running this command: 

```sh
$ kubectl get all -n easytravel
```

![validation](assets/validate.png)

## Access the Sock Shop web app

The application deployment created a <b>Service</b> resource of type <b>Load Balancer</b> to expose the <b>front-end</b> and <b>carts</b> services to the public internet. It might take a few minutes before the public IPs become available.
You can obtain the app URLs by running this script:

```sh
$ ./get-sockshop-urls.sh -istio
```
The script will wait until the IPs are available and will then print those. 

![app urls](assets/app_urls.png)

Click on the Production or Dev frontend URL to load the Sock Shop home page.

The URLs are stored in the `sockshop.conf` file. You can always get those by running this command (from the current directory):

```sh
$ cat configs.txt
```

## Create Sock Shop user accounts

This following script will create a few user accounts that will be used by the synthetic monitors to generate traffic on the Production environment:

```sh
$ ./create-sockshop-accounts.sh
```

## Explore the app

Load the Sock Shop app page in your browser.

![sockshop](assets/sockshop.png)

Play around! 

Run some transactions from the browser (Register, Logout, Login, Catalogue, Add to Cart, etc).

You can manually register a new account or log in with one that was created during the previous step:

`username : perform`

`password : 1234`

<b><u>NOTE</u></b>: The checkout service in the application is not currently implemented. You can add items to the shopping cart but you will not be able to checkout.

---

:arrow_forward: [Next : #2 Deploy the OneAgent Operator](../02_Deploy_OneAgent_Operator)

:arrow_up_small: [Back to overview](../)
