# Pre-requisites

If you're deploying a cluster with kube-aws:

* [EC2 instances whose types are larger than or equal to `t2.medium` should be chosen for the cluster to work reliably](https://github.com/kubernetes-incubator/kube-aws/issues/138)
* [At least 3 etcd, 2 controller, 2 worker nodes are required to achieve high availability](https://github.com/kubernetes-incubator/kube-aws/issues/138#issuecomment-266432162)

## Deploying to an existing VPC

`kube-aws` tries its best to not modify your existing AWS resources. It's our users' responsibility to ensure existing AWS resources provided to `kube-aws` are properly configured.

Please note that you don't need to care about modifications if you've instructed `kube-aws` to create all the AWS resources for you i.e. you've omitted `vpcId` and `routeTableId` from `cluster.yaml`.

What `kube-aws` does modify are:

* Adding a record set for Kubernetes API Endpoint to an existing hosted zone you've provided via the `hostedZoneId` configuration key in `cluster.yaml`
* Adding one or more subnet(s) to an existing VPC specified by the `vpcId`
* Associating one or more subnet(s) to an existing route table specified by the `routeTableId`

See [`cluster.yaml`](https://github.com/kubernetes-incubator/kube-aws/blob/master/core/controlplane/config/templates/cluster.yaml) for more details.

All the other configurations for existing AWS resources must be done properly by users before `kube-aws` is run.

For example, if you're deploying a cluster to an existing VPC:

* An internet gateway or a NAT gateway needs to be added to VPC before cluster can be created
  * Or [all the nodes will fail to launch because they can't pull docker images or ACIs required to run essential processes like fleet, hyperkube, etcd, awscli, cfn-signal, cfn-init.](https://github.com/kubernetes-incubator/kube-aws/issues/120)
* Existing route tables must have a route to Internet in some form. For example, a default route to an internet gateway or to a NAT gateway via `0.0.0.0/0` would be needed or your cluster won't come up. See [a relevant issue about it](https://github.com/kubernetes-incubator/kube-aws/issues/121#issuecomment-266255407).
* Existing route tables and/or subnets to be reused by kube-aws must be tagged with the key `kubernetes.io/cluster/$CLUSTER_NAME` and "shared" as a value.
  * Or [Kubernetes will fail to create ELBs correspond to Kubernetes services with `type=LoadBalancer`](https://github.com/kubernetes-incubator/kube-aws/issues/135)
* ["DNS Hostnames" must be turned on before cluster can be created](https://github.com/kubernetes-incubator/kube-aws/issues/119)
  * Or etcd nodes are unable to communicate each other thus the cluster doesn't work at all

Once you understand pre-requisites, you are [ready to launch your first Kubernetes cluster][aws-step-1].

[aws-step-1]: kubernetes-on-aws.md
[aws-step-2]: kubernetes-on-aws-render.md
[aws-step-3]: kubernetes-on-aws-launch.md
[aws-step-4]: kube-aws-cluster-updates.md
[aws-step-5]: kubernetes-on-aws-node-pool.md
[aws-step-6]: kubernetes-on-aws-add-ons.md
[aws-step-7]: kubernetes-on-aws-destroy.md
