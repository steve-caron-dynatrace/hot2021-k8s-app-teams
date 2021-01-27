## Exercise #10 Blue-Green Deployment

### That Promo Campaign again!

<u>Scenario</u>: So you averted a major disaster with the help of Dynatrace when you tested the new cart feature for the SockShop promo campaign...

But marketing is returning in force. :grinning: 

The promo campaign release has been shortly delayed and the issue quickly duct-taped. Your demands for more testing time are rejected!

![test_in_production](../../assets/images/test-in-production.jpg)

Fortunately, you have a strategy.

### Service Mesh at the rescue

The Kubernetes platform team has recently deployed Istio, a service mesh, on your Kubernetes cluster. Istio provides dynamic traffic management capabilities, without any code change.

You managed to convince the Sock Shop management to not expose the new <b>carts-v2</b> service to all customers. Initially, it will only be available for non-customers (users who are not logged in). Existing customers will stay on the current version <b>carts-v1</b>. The frontend was also cosmetically changed, depending the version, the header color will vary : v1 is in grey, v2 is in blue.

### Let's deploy that v2!

Go in the web terminal, make sure your are in the `exercises` directory. Execute the following:

```sh
$ ./toggle-sockshop-promo-ff.sh
```


![promo_socks](assets/promo_socks.png)

- The deployment mode for this promo campaign is using a feature flag
- By default, the promo is disabled but it can be enabled on the fly using an API call

The day for the release of the promo is coming soon... 

But you managed to win your point and get a bit of time to do some testing! And Dynatrace can help!

### Turn on the promo campaign in dev

Enable the promo feature by running the following command and enter `1` at the prompt :

```sh
$ ./toggle-sockshop-promo-ff.sh
```


### Deploy new carts build in dev

Also, in parallel, there are changes coming again to the <b>carts</b> services. The dev team is working on some new feature for marketing.

Execute the following command to deploy a new <b>carts</b> build in dev:

```sh
$ ./deploy-carts-new-build.sh
```

## Problems?

<u>Keep an eye on the Slack channels and your emails</u> ... at one point you will receive one or two emails from the "Dynatrace team"

![email_notification_cpu](assets/email_notification_cpu.png)

- You can click on the Open in browser link or look at <b>Problems</b> in the Dynatrace menu

- We see 2 hosts are experiencing CPU saturation

    ![cpu_saturation_problems](assets/cpu_saturation_problems.png)

- We see the same by looking at <b>Kubernetes</b> view which shows 2 problematic <b>nodes</b>

    ![kubernetes_problematic_nodes](assets/kubernetes_problematic_nodes.png)

- Click on <b>Analyze nodes</b> (bottom right) to get more details on the nodes. We see 2 nodes are at 99% CPU utilization
  
    ![kubernetes_node_analysis](assets/kubernetes_node_analysis.png)

- Drill-down into one of these 2 nodes to get to the <b>Host</b> view. 
- Look at <b>CPU Usage</b>

    ![host_cpu_usage](assets/host_cpu_usage.png)

- Click on <b>Consuming processes</b>. The processes (containers) running on the host (node) will be listed. Sort them by `CPU usage` and you will see the culprit, the "hot new service" container. 

    ![host_consuming_processes](assets/host_consuming_processes.png)

- This issue had the potential to create bigger problems impacting the whole cluster and the rest of the workload. The pod definition in this case does not have any resource limit. But the service itself needs to be fixed and until then, we will remove this from the cluster.

    ```sh
    $ ./undeploy-hot-new-service.sh
    ```

- The CPU consumption on the nodes will go back to normal. Within a few minutes you will receive a `RESOLVED Problem` email notification.


If you look back at Slack, you should have received notifications on the `#carts-support-ops` channel; if not, wait a bit.
  
  ![slack_cart_support_ops_problem_open](assets/slack_carts_support_ops_problem_open.png)

- Click on the link to drill-down to the <b>Problem</b> details

    ![carts_promo_problem](assets/carts_promo_problem.png)

- The <b>carts</b> service is experiencing a significant transaction failure rate increase, which is always bad but even worse this time because we are in the promo campaign. :rage: 
- Getting back into Dynatrace
  - You can look drill-down to <b>Analyze failure rate degradation</b>

    ![carts_promo_failure_analysis](assets/carts_promo_failure_analysis.png)

    - You can see the error message: something is not implemented right in the promo campaign... But this is affecting the whole <b>carts</b> service and no customer can add items to the cart anymore... This is bad... :grimacing:
    - Good thing we had the promo campaign deployed using a feature flag! We can easily turn off the flag and things should go back to normal.
      - Execute the following command,  entering `2` at the prompt

        ```sh
        $ ./toggle-sockshop-promo-ff.sh
        ```
    - After a few minutes, you will get a message in the #carts-support-ops channel notifying the problem is resolved
  
        ![slack_carts_support_ops_problem_resolved](assets/slack_carts_support_ops_problem_resolved.png)

So Dynatrace helped the Kubernetes platform team to identify a rogue new service and remove it before it causes issues to the rest of the cluster workload.

It also helped to quickly solve a potentially catastrophous problem with the promo campaign.

What about the new build that the dev team is working on? We don't want to repeat past mistakes and deploy a faulty build to production... :unamused:

- Look at the `#carts-dev` Slack channel, there should be a problem notification (if not wait a bit).

    ![slack_carts_dev_problem_open](assets/slack_carts_dev_problem_open.png)

- Seems the currently tested new build is expriencing performance issues. Click on the problem link to drill down into the details.

    ![carts_dev_problem](assets/carts_dev_problem.png)

- Click on <b>Analyze response time degradation</b>

    ![carts_dev_response_time_analysis](assets/carts_dev_response_time_analysis.png)

- What we see here:
  1. The affected request is `addToCart`
  2. The response time for that request is around 9.3 seconds
  3. Top findings tell us that it is essentially all <b>Active wait time</b>
  4. Let's get down to code level by clicking on <b>View method hotspots</b>

    ![carts_dev_method_hotspots](assets/carts_dev_method_hotspots.png)

- What we see here:
  1. Click on <b>Hotspots</b> to switch from the call hierarchy to the hotspot view
  2. Select the first method `Thread.sleep`. 
     - You will see below the reverse method call hierarchy
  3. Click on `expand`
     - You see the `Thread.sleep` method is being called by the `ItemsControllers.addToCart` method

This is the information the dev team need to fix the issue in the code. 

You can also get further down into deep dive by clicking on the drilldown to PurePaths button (top right)

![drill_down_to_PurePaths](assets/drill_down_to_PurePaths.png)

Click on one of the PurePaths listed. You can then look at the PurePath details, which are showing the calls to the MongoDB data store and also that the `addToCart` method is where the time is spent. 

![carts_dev_purepath](assets/carts_dev_purepath.png)

Since this build is bad, you will roll back to the previous build by executing the following commands:

```sh
$ kubectl rollout history deployments carts -n dev
```

This will show you the deployment rollout story. Normally you should have two deployments. 

![carts_dev_rollout_history](assets/carts_dev_rollout_history.png)

That means you need to rollback to `Revision 1`

```sh
$ kubectl rollout undo deployments carts --to-revision=1 -n dev
```
It will take about 5 minutes for the new carts pod to become ready.

Eventually, you will receive a Slack message in the `#carts-dev` channel notifying the problem is resolved.

![slack_carts_dev_problem_resolved](assets/slack_carts_dev_problem_resolved.png)

<b><u>ONE LAST THING</u></b>You should disable or delete your alerting profiles after this class if you don't want to continue receive alert emails.

---

[Previous : #9 Configure k8s cluster monitoring integration](../09_Configure_k8s_cluster_monitoring_integration) :arrow_backward: 

:arrow_up_small: [Back to overview](../)