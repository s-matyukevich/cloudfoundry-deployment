#Cloud Foundry Deployment to AWS

Ther repository includes  set of Cloud Formation templates to deploy Cloud Foundry on AWS.
The deployment is an example of a minimalistic deployment of Cloud Foundry, including all crucial components for its basic funcionality. It allows you to deploy Cloud Foundry for educational purposes, so you can poke around and break things.

IMPORTANT: This is not meant to be used for a production level deployment as it doesn't include features such as high availability and security.

###Prerequisites:
- aws cli
- env variables (copy from .envrc-example to .envrc and source it)

##How To Use:

1. Setup environment variables. Use .envrc-example file as an example.
2. To deploy CF to AWS:
```
$scripts/stack.sh create
```

After a while you will get running CF deployment including:
- EC2 key pair (stored as bosh.pem)
- VPC with NAT and JumpBox instances
- (micro)BOSH instance
- CF deployment (we use aws_minimal CF manifest from cf-release) like:
```
+------------------------------------+---------+---------------+-------------+
| Job/index                          | State   | Resource Pool | IPs         |
+------------------------------------+---------+---------------+-------------+
| api_z1/0                           | running | small_z1      | 10.0.16.4   |
| doppler_z1/0                       | running | small_z1      | 10.0.16.6   |
| etcd_z1/0                          | running | small_z1      | 10.0.16.104 |
| ha_proxy_z1/0                      | running | small_z1      | 10.0.0.11   |
|                                    |         |               | PUB_IP_ADDR |
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

The script generates RSA key pair so to access JumpBox run:
```
$ssh -i bosh.pem ubuntu@PUB_IP_ADDRESS_OF_JUMPBOX
```
To check your CF deployment:
```
# PUB_IP_ADDR is the address of load balancer.
cf login -u admin -p PASSWORD -a api.PUB_IP_ADDR.xip.io --skip-ssl-validation
cf create-space dev && cf target -o default_organization -s dev
git clone https://github.com/cloudfoundry/cf-acceptance-tests.git && cd cf-acceptance-tests/assets/dora
cf push dora && cf logs dora --recent
```

##Deletion of stack:

IMPORTANT: Before deletion of stack do not forget to delete (from JumpBox):
- CF deployment
```$bosh -n delete deployment cf```
- micro bosh instance
```$cd /home/ubuntu/my-bosh && bosh-init delete micro.yml```

###WIP:
- add Diego
  - add mappings of releases (BOSH - CF - Diego)

###TODOs:
- secure deployment (SG, ACLs, generated passwords, keys, certificates etc)
- use spot instances
- add .Net cell to Diego

####Useful links:

Diego Windows Release - https://github.com/cloudfoundry-incubator/diego-windows-release
Garden Windows Release - https://github.com/cloudfoundry-incubator/garden-windows-release
Generating SSL certificate - http://docs.pivotal.io/pivotalcf/customizing/self-signed-ssl.html
http://www.markkropf.com/blog/2015/5/17/setup-lattice-to-run-your-windows-apps
