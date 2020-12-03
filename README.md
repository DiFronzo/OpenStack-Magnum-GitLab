# OpenStack Magnum with deplyment of GitLab CI/CD
Deployment of a Kubernetes Cluster and GitLab CI/CD
```
Prerequisites:
- Docker
- Docker Compose
- Python 3
 - python-magnumclient
 - python-openstackclient
```
You also need basic understanding of OpenStack Heat Templates and Docker.

# Basic setup instructions

## Install dependencies
```BASH
git clone %THIS PROJECT%
cd %THIS PROJECT%
pip3 install -r requirements.txt
```
Installation of Docker is found [her](https://docs.docker.com/engine/install/ubuntu/) and Docker Compose is found [her](https://docs.docker.com/compose/install/)  for Linux.

## Run the cluster

First you will need to log in to your OpenStack infrastructure and download the [openrc file](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_openstack_platform/7/html/command-line_interface_reference_guide/ch_cli#openrc-dashboard).

```BASH
source openrc.sh #source your own openrc from OpenStack
```
Next edit `iac_cluster_env.yaml` and enter name of your `keypair` and `external_network`. You don't have a keypair? Look at [this guide](https://docs.openstack.org/python-openstackclient/pike/cli/command-objects/keypair.html). You will also need to edit parameters in `iac_cluster.yaml` for `flavor`, `node_count`, and etc.

Once you have finished the steps above, you can now deploy the stack. Change `<name-of-stack>` with a name of your choosing.
```BASH
openstack stack create <name-of-stack> -t iac_cluster.yaml -e iac_cluster_env.yaml
```
Now wait until the cluster has the status "CREATE_COMPLETE"
```bash
openstack coe cluster list
```
The next step is to configure the newly created Kubernetes cluster, this needs **Docker** and **Docker Compose** to run locally.
```bash
vim .env
```
The file should look something similar to the following with URL and TOKEN from your prefered [gitlab](https://docs.gitlab.com/runner/register/#requirements) and follow "project-specific runner". Add the UUID or name of the cluster in "CLUSTER=" (workes best with UUID (use `openstack coe cluster list` to find this). 
```bash
# .env.compose
URL=https://gitlab.com/
TOKEN=
CLUSTER=
```
Then add env. variables for OpenStack access. This include you password in clear text! Do not upload this file anywhere!
```bash
(env | grep OS_) >>.env
```
Last step is to build and run docker-composer to establish a connection between Kubernetes and GitLab.
```bash
docker-compose build
docker-compose up
```

# Usage (Kubernetes)

When the cluster is created you can do the following to connect.
```bash
mkdir -p ~/clusters/kubernetes-cluster
$(openstack coe cluster config <your-cluster> --dir ~/clusters/kubernetes-cluster)
kubectl get nodes
```

If lost connection:
```bash
export KUBECONFIG=~/clusters/kubernetes-cluster/config
```

### Delete everything created for gitlab-runner

If you want to remove everything connected to gitlab-runner.
```bash
kubectl delete deployment,configmap,rolebinding,role,serviceaccount --all --namespace=gitlab-runner
kubectl delete namespace gitlab-runner
```

To add the configuration again. You will have to first remove your token from `gitlab-runner-k8s.yaml` so, it look like this `token =`.
```bash
docker-compose build
docker-compose up
```

### Scale nodes

To change the number of nodes in your cluster, you can do the following:
```bash
openstack coe cluster update <your-cluster> replace node_count=<N>
```

# Scale GitLab runners

To change the number of pods in your cluster, you can change the following line in the YAML template (`gitlab-runner-k8s.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-runner
  namespace: gitlab-runner
spec:
  replicas: X
```

To change the amount of requested or limit the amount of the gitlab-runners, you can change or add the following line(s) in the YAML template. Further information on [Kubernetes.io](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-runner
  namespace: gitlab-runner
...
...
          image: gitlab/gitlab-runner:latest
          imagePullPolicy: Always
          name: gitlab-runner
          resources:
            requests:
              cpu: "X"
              memory: "X"
            limits:
              cpu: "X"
              memory: "X"
```

# About the project

## What is this repo actually doing ?
This repo is for easy deplyment of a Kubernetes Cluster and for adding a connection to a GitLab repo CI/CD.

Please see the road map section of the README.md to see upcoming functionality and releases of the module.

## Limitations
There is a [bug](https://storyboard.openstack.org/#!/story/2008383) in OpenStack Magnum that does not allow activating "Docker Registry" in the Heat Template.

## Road Map (For this repo)
Here is the functionality that is on the road map for this repo
 - [ ] Overwrite the token in `gitlab-runner-k8s.yaml` when running Docker Compose.


