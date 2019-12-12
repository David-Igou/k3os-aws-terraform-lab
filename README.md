# k3os-aws-terraform

The purpose of this repo is to deploy k3os on AWS, and defining configuration via terraform variables. The current design is meant to be light weight with limited features, but very easy to bring up.

<!-- TODO: Add links to everything used in this project -->

Currently this only supports a single master deployment.

The main repo https://github.com/David-Igou/k3os-aws-terraform will have more features added, but this exists as a route to the easiest/cheapest multi-worker clusters in the cloud possible.

Future pushes will add AWS plugins, db options, loadbalancer options, more k3s flags, and hopefully I'll find an easy way to automatically deploy manifests post install.

## Limitations

To keep the barrier of entry low, currently this onle does a single master `n` worker setup. The most you can really configure here is adding an eip (So the master keeps the same ip between rebuilds).

Configurations that are not supported:

* Availability zones - This might come later, but the shortest path to working storage is EBS which is locked to AZs anyway.
* External database - One master so, you're using local sqlite
* ELB - One master, once again.

Features I'll eventually add to this repo:

* Automatically generating the node tokens
* More intricate OOTB configurations via user_data (read on the k3os config.yaml)
* `variables.tf` is soul-crushingly badly put together

Full leveraging of AWS will come to the parent repo over time

## Common changes

1.  "I don't want traefik/metrics-server or another feature k3s ships with"

* This can be disabled via --no-deploy, which you can pass using k3_args

2.  "Should I put tons of things in `manifests`"

*  I don't - I put the CSI deployments, and a GitOps operator in mine, and have the operator pull everything else I plan on running down

## Bootstrapping the cluster

k3s automatically creates manifests stored in `/var/lib/rancher/k3os/server/manifests/` - When `sync_manifests` is set to true, a provisioner will sync the local manifests directory to the remote.

This also pulls the kubeconfig down, but you'll have to edit the url from 127.0.0.1 to whatever resolves to your master.

## Idle Performance:

YMMV by what you run, of course, but here are the metrics-server CPU/Memory consumption post install (no loads running). (k3s 1.0.0, k3os 0.7.1)


### All t2.micros:
```
[~]# kubectl top nodes
NAME                                       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-10-1-1-116.us-east-2.compute.internal   12m          1%     245Mi           24%       
ip-10-1-1-217.us-east-2.compute.internal   11m          1%     221Mi           22%       
ip-10-1-1-232.us-east-2.compute.internal   32m          3%     571Mi           58%       
ip-10-1-1-51.us-east-2.compute.internal    9m           0%     222Mi           22% 
```

### t2.micro master, t2.nano computes
```
[~]$ kubectl top nodes
NAME                                       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
ip-10-1-1-172.us-east-2.compute.internal   10m          1%     228Mi           47%
ip-10-1-1-202.us-east-2.compute.internal   30m          3%     590Mi           60%
ip-10-1-1-237.us-east-2.compute.internal   10m          1%     225Mi           47%
ip-10-1-1-246.us-east-2.compute.internal   12m          1%     197Mi           41%
```


<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| agent\_image\_id | AMI to use for k3s agent instances | string | `"ami-0ed92ab0a9ecbbcf4"` | no |
| agent\_instance\_type |  | string | `"t2.micro"` | no |
| agent\_node\_count | Number of agent nodes to launch | number | `"3"` | no |
| api\_eip | EIP Association id for the master node | string | `"null"` | no |
| data\_sources | data sources for node | list | `[ "aws" ]` | no |
| dns\_nameservers | kernel modules for node | list | `[ "8.8.8.8", "1.1.1.1" ]` | no |
| k3s\_args | Additional k3s args (kube-proxy, kubelet, and controller args also go here | list | `[]` | no |
| k3s\_cluster\_secret | Override to set k3s cluster registration secret - This will be made random at default | string | `"abcdef12345"` | no |
| kernel\_modules | kernel modules for node | list | `[]` | no |
| keypair\_key | Keypair Key | string | `"ssh-rsa AAAAB3NADSKJFJDSAFdsafds example@example.com"` | no |
| keypair\_name | Keypair name | string | `"k3s_key"` | no |
| ntp\_servers | ntp servers | list | `[ "0.us.pool.ntp.org", "1.us.pool.ntp.org" ]` | no |
| server\_image\_id | AMI to use for k3s server instances | string | `"ami-0ed92ab0a9ecbbcf4"` | no |
| server\_instance\_type |  | string | `"t2.micro"` | no |
| server\_node\_count | Number of server nodes to launch | number | `"1"` | no |
| ssh\_keys | SSH Keys to inject into nodes | list | `[]` | no |
| sync\_manifests | If true, terraform will copy the contents of the `manifests` directory in the repo to /var/lib/rancher/k3s/server/manifestsi - ssh-agent required | bool | `"false"` | no |
| sysctls | sysctl params for node | list | `[]` | no |
| vpc\_cidr | VPC CIDR | string | `"10.0.0.0/16"` | no |
| vpc\_subnet | VPC Subnet | string | `"10.0.1.0/24"` | no |

## Outputs

| Name | Description |
|------|-------------|
| public\_api\_ip |  |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->



