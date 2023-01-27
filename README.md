[![Try Knative Logo](https://static.codingforentrepreneurs.com/media/courses/knative-serverless-kubernetes/df87329c-3d26-4358-b913-5ed037dabd8b.jpg)](https://www.codingforentrepreneurs.com/courses/try-knative/)

# Try Knative
This repository is the reference code for our in-depth course [Try Knative](https://www.codingforentrepreneurs.com/courses/try-knative/) along with the related article on [our blog](https://www.codingforentrepreneurs.com/blog/try-knative-on-lke).

There are a few requirements before you get started.

## Requirements
- Watch [Terraforming Kubernetes on Linode](https://www.codingforentrepreneurs.com/courses/terraforming-kubernetes-on-linode/) or have experience with Kubernetes clusters
- [Git](https://git-scm.com/downloads) installed
- [Terraform](https://developer.hashicorp.com/terraform/downloads) installed
- [Kubectl](https://kubernetes.io/docs/tasks/tools/) installed

If you have all of these complete, let's do this.

## 1. Clone this repo
I have modified the [Terraforming Kubernetes Rapid-Fire](https://github.com/codingforentrepreneurs/terraforming-kubernetes-rapid) repo for this project. I removed the `git` history and `k8s.yaml` file.

```
git clone https://github.com/codingforentrepreneurs/try-knative
cd try-knative
```
Once you clone this repo, you'll have a `installers/install-knative.sh` file that will install Knative Serving and Istio on your Kubernetes cluster based on their standard installation instructions. More on this in a bit.

If you want to start fresh, checkout the `fresh` branch and remove the git history

```
git checkout fresh
rm -rf .git
git init
```
Now you have a fresh repo that includes only the code for the rest of this README.

## 2. Initialize Terraform
Create an account on [Linode](https://www.linode.com/cfe) and get an API Key in your linode account [here](https://cloud.linode.com/profile/tokens).

Once you have a key, do the following:

```bash
echo "linode_api_token=\"YOUR_API_KEY\"" >> terraform.tfvars
echo "k8s_node_type=\"g6-standard-2\"" >> terraform.tfstate
```

Through my tests, the minimum node instance type we need to use for Knative is `g6-standard-2` with a minimum of 3 nodes in the cluster.

Now run
```bash
terraform init
```
Be sure that if you're using _git_ that you have at least [this .gitignore](./blob/main/.gitignore) file in your repo.


## 3. Add Autoscaling to your Kubernetes Cluster

In `infra.tf`, we'll update the `linode_lke_cluster.terraform_k8s` resource to add autoscaling to our cluster and update the `k8s_version` to `1.25`.

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version="1.25"
    label="try-knative"
    region="us-east"
    tags=["try-knative"]
     pool {
        type  = var.k8s_node_type
        count = 3
        autoscaler {
            min = 3
            max = 8
        }
    }
}
```

The `autoscaler` declaration will ensure that Kubernetes has as many nodes as it needs to run your containerized applications. LKE will manage this for you without further intervention. The _autoscaler_ has strange behavior in Terraform so we'll update it in a bit.

_Optional_: I also updated the `label`, and `tags` to be `try-knative` instead of `terraform-k8s` for this project. You should update them as you see fit. I left the name of Terraform resource `linode_lke_cluster` as `terraform_k8s` in order to limit how much of `infra.tf` we need to change. 


## 4. Terraform your Kubernetes Cluster

```bash
terraform apply
```
> Use `terraform apply -auto-approve` if you're really in a hurry.


## 5. Update Terraform Lifecycle to ignore Autoscaling Changes

Adding the autoscaling declaration in our LKE resource is a bit wonky because Terraform  will _constantly_ try to update your cluster to the declare document state.

For example, if LKE autoscaled your cluster to 5 nodes, Terraform will try to update your cluster to 3 nodes. To fix this, we can just _ignore_ changes to the `pool` declaration until we want to explicitly change it.

```hcl
resource "linode_lke_cluster" "terraform_k8s" {
    k8s_version="1.24"
    label="terraform-k8s"
    region="us-east"
    tags=["terraform-k8s"]
     pool {
        type  = var.k8s_node_type
        count = 3
        autoscaler {
            min = 3
            max = 8
        }
    }
    lifecycle {
        ignore_changes = [
            pool,
        ]
        create_before_destroy = true
    }
}
```


## 6. Install Knative Serving and Istio

Knative Serving is the core Kubernetes component that allows you to run serverless containerized applications. Istio is a service mesh that allows you to manage traffic between your applications and, for our case, with the outside world.

You can install them on these links _or_ our bash script below:

- Install [Knative Serving Docs](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#install-the-knative-serving-component)
- Install [Knative + Istio Installation Docs](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#install-a-networking-layer)


The macOS/Linux bash script is avaiable in this repo at: [installers/install-knative-istio.sh](./blob/main/installers/install-knative-istio.sh). Let's run it with now:

```bash
chmod +x ./installers/install-knative-istio.sh
./installers/install-knative-istio.sh
```

At this time, it's recommended that Windows users use the links for _Knative Serving_ and _Knative + Istio_ installation from above.


## 7. Confirm Installation

When you run `./installers/install-knative-istio.sh`, you should see the output related to your knative/istio installation. Here's the command we can run again to get our Knative Ingress IP address. If you're using Linode and LKE, this IP Address is configured from a Linode load balancing service called NodeBalancers which is how we can have Kubernetes provide us another IP Address.

```bash
kubectl --namespace istio-system get service istio-ingressgateway
export KNATIVE_INGRESS_IP=$(kubectl --namespace istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo "Your IP Address is: $KNATIVE_INGRESS_IP"
echo "Add a cname record for your domain using the above IP address."
```

## 8. Knative Installed!

At this point, I recommend you review our [Try Knative article](https://www.codingforentrepreneurs.com/blog/try-knative-on-lke) on deploying containers with Knative services, mapping domains, and more!