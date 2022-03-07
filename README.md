# Step 0 "Kubernetes"

In order to install and work with Linkerd, you need a Kubernetes cluster to work with. (Of course!)

I am using [minikube](https://minikube.sigs.k8s.io/docs/start/) on Fedora 35 (and windows).

But feel free to use which ever alternative you have.

## Step 1 "CLI Installation"

Download and install the linkerd CLI onto your local machine.

```bash
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
```

*Be sure to follow the instructions to add it to your **path.***

Once Installed, verify the CLI is installed correctly with:

```bash
linkerd version
```

You should see the CLI version, and also Server version: unavailable. This is because you haven’t installed the control plane on your cluster. Don’t worry—we’ll fix that soon enough.

## Step 2 "Validate the Kubernetes Cluster"

Before we can install the Linkerd control plane, we need to check and validate that everything is configured correctly. To check that your cluster is ready to install Linkerd, run:

```bash
linkerd check --pre
```
If there are any checks that do **not pass**, make sure to follow the provided links and fix those issues before proceeding, or ask for help!


## Step 3 "Install the control plane onto the cluster"
Now that we have the CLI running locally and a cluster that is correct and ready to go, we can install the control plane, do this by running:

```bash
linkerd install | kubectl apply -f -
```
The linkerd install command generates a Kubernetes manifest with all the core control plane resources (feel free to inspect this output if you’re curious). Piping this manifest into kubectl apply then instructs Kubernetes to add those resources to your cluster!

It may take a minute or two for the control plane to finish installing. Wait for the control plane to be ready (and verify your installation) by running:

```bash
linkerd check
```

## Step 4 "Install the Demo App!"

Congratulations, Linkerd is installed! However, it’s not doing anything just yet. To see Linkerd in action, we’re going to need an application.

Let’s install a demo application called Emojivoto. Emojivoto is a simple standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the user to vote on their favorite emojis.

Install Emojivoto into the emojivoto namespace by running:

```bash
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -
```
This command installs Emojivoto onto your cluster, but Linkerd hasn’t been activated on it yet—we’ll need to “mesh” the application before Linkerd can work its magic.

Before we mesh it, let’s take a look at Emojivoto in its natural state. We’ll do this by forwarding traffic to its web-svc service so that we can point our browser to it. Forward web-svc locally to port 8080 by running:

```bash
kubectl -n emojivoto port-forward svc/web-svc 8080:80
```
Now visit http://localhost:8080. Voila! You should see Emojivoto in all its glory.

If you click around Emojivoto, you might notice that it’s a little broken! For example, if you try to vote for the donut emoji, you’ll get a 404 page. Don’t worry, these errors are **intentional**. (This is covered in another guide.)

With Emoji installed and running, we’re ready to mesh it—that is, to add Linkerd’s data plane proxies to it. We can do this on a live application without downtime, thanks to Kubernetes’s rolling deploys. Mesh your Emojivoto application by running:

```bash
kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```
What this command does:

- Retrieves all of the deployments running in the emojivoto namespace. 
- Runs their manifests through linkerd inject and then reapplies it to the cluster. 

(The linkerd inject command simply adds annotations to the pod spec that instruct Linkerd to inject the proxy into the pods when they are created.)

As with install, inject is a pure text operation, meaning that you can inspect the input and output before you use it. Once piped into kubectl apply, Kubernetes will execute a rolling deploy and update each pod with the data plane’s proxies.

Congratulations! You’ve now added Linkerd to an application! Just as with the control plane, it’s possible to verify that everything is working the way it should on the data plane side. Check your data plane with:

```bash
linkerd -n emojivoto check --proxy
```

And, of course, you can visit [localhost](http://localhost:8080) (http://localhost:8080) and once again see Emojivoto in all its meshed glory.

## Step 5 "Explore!"

In order to get a better "feel" of what Linkerd does, we can install an *extension* that installs a nice looking dashboard to Linkerd.

To do this, we need the viz extension, which will install an on-cluster metric stack and dashboard.

To install viz, run the following:

```bash
linkerd viz install | kubectl apply -f - # install the on-cluster metrics stack
```
Once installed, we will validate everything is okay by running:

```bash
linkerd check
```

If everything looks good, we can access the dashboard with the following command:

```bash
linkerd viz dashboard &
```

## Step 6 "That's it! (Time for beers?)"

Well done, you've installed linkerd on a cluster and had a look at the cool metrics dashboard. If you'd like more information on Linkerd, and potentially debugg the issue during **Step 4** you can check out the following links:

- Learn how to use Linkerd to debug the errors in [Emojivoto](https://linkerd.io/2.11/tasks/debugging-your-service/).
- Learn how to [add your own services](https://linkerd.io/2.11/adding-your-service/) to Linkerd without downtime.
- Learn how to install other [Linkerd extensions](https://linkerd.io/2.11/tasks/extensions/) such as Jaeger and Buoyant Cloud.
- Learn more about [Linkerd’s architecture](https://linkerd.io/2.11/reference/architecture/)
- Learn how to set up [automatic control plane mTLS credential rotation](https://linkerd.io/2.11/tasks/automatically-rotating-control-plane-tls-credentials/) for long-lived clusters.

## Documentation can be found here

- Getting [started](https://linkerd.io/2.11/getting-started/) with Linkerd
- Official [Documentation](https://linkerd.io/2.11/overview/)
