Cloud Foundry Deployment to AWS
===============================

Set of Cloud Formation templates to deploy Cloud Foundry to AWS.

Prerequisites:
- aws cli
- env variables (copy from .envrc-example to .envrc and source it)

How To Use:

1. Setup environment variables. Use .envrc-example file as an example.
2. To deploy CF to AWS:
```
$scripts/stack.sh create
```

After a while you will get running CF deployment including:
- EC2 key pair (stored as bosh.pem)
- VPC with NAT and JumpBox instances
- microbosh instance
- CF deployment (aws_minimal CF template used) like:
```
+------------------------------------+---------+---------------+-------------+
| Job/index                          | State   | Resource Pool | IPs         |
+------------------------------------+---------+---------------+-------------+
| api_z1/0                           | running | small_z1      | 10.0.16.4   |
| doppler_z1/0                       | running | small_z1      | 10.0.16.6   |
| etcd_z1/0                          | running | small_z1      | 10.0.16.104 |
| ha_proxy_z1/0                      | running | small_z1      | 10.0.0.11   |
|                                    |         |               | 52.3.105.39 |
| hm9000_z1/0                        | running | small_z1      | 10.0.16.5   |
| loggregator_trafficcontroller_z1/0 | running | small_z1      | 10.0.16.7   |
| nats_z1/0                          | running | small_z1      | 10.0.16.103 |
| nfs_z1/0                           | running | small_z1      | 10.0.16.105 |
| postgres_z1/0                      | running | small_z1      | 10.0.16.101 |
| router_z1/0                        | running | small_z1      | 10.0.16.102 |
| runner_z1/0                        | running | small_z1      | 10.0.16.9   |
| stats_z1/0                         | running | small_z1      | 10.0.16.10  |
| uaa_z1/0                           | running | small_z1      | 10.0.16.8   |
+------------------------------------+---------+---------------+-------------+
```
To check status of your deployment and to find IP address of JumpBox and CF's load balancer use AWS console (in EC2 or Cloud Formation) or run
```
$scripts/stack.sh describe
```

To access JumpBox:
```
$ssh -i bosh.pem ubuntu@IP_ADDRESS_OF_JB
```
To check your CF deployment:
```
$cf login -u admin -p PASSWORD -a api.IP_OF_LB.xip.io --skip-ssl-validation
$cf create-space dev && cf target -o default_organization -s dev
$git clone https://github.com/cloudfoundry/cf-acceptance-tests.git && $cd cf-acceptance-tests/assets/dora
$cf push dora && cf logs dora --recent
```

Deletion of stack:

IMPORTANT: Before deletion of stack do not forget to delete (from JumpBox):
- CF deployment
```$bosh -n delete deployment cf```
- micro bosh instance
```$cd /home/ubuntu/my-bosh && bosh-init delete micro.yml```


TODOs:
- secure deployment (SG, ACLs, generated passwords, etc)
- use spot instances
- add Diego
- add .Net cell to Diego

Useful links:

http://www.markkropf.com/blog/2015/5/17/setup-lattice-to-run-your-windows-apps
