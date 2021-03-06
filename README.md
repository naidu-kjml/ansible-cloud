# ansible_cloud
Ansible Playbooks for setting up cloud services such as Mesos, Docker and Kubernetes.

Most of the stuff for Marathon, Mesos and Zookeeper is copied from the [AnsibleShipyard](https://github.com/AnsibleShipyard) repositories with few or no tweaks.

To try it out do:

1. Configure `~/.boto` as described in [Boto Config](http://boto.readthedocs.org/en/latest/boto_config_tut.html):

   ```
   [Credentials]
   aws_access_key_id = <your_access_key_here>
   aws_secret_access_key = <your_secret_key_here>
   ```
1. Copy the `ec2.ini`file to `/etc/ansible/ec2.ini`.
1. Add your `*.pem` file to the `ssh-agent`:

   ```
   $ ssh-agent bash 
   $ ssh-add ~/.ssh/keypair.pem 
   ```
1. The [AWS Command Line Interface](https://aws.amazon.com/cli/) must be installed, configured and the `aws` executable be on the path. Used (until Ansible 2.2 is released) to create the NAT gateway.
1. Run the playbook as `ansible-playbook -i ec2.py main.yml` to set up the EC2 instances.
1. Run the playbook as `ansible-playbook -i ec2.py provision_cluster.yml` to provision the instances with Mesos, Docker, Zookeeper etc.

## Environment variables
The following environment varibles can be configured for Ansible lookup

### AWS
* AWS_EC2_KEY_PAIR: the name of the key pair to use when provisioning the cluster (EC2 instances etc.).
* AWS_EC2_LB_CERT: the `arn` resource for the AWS ELB certificate.
* AWS_S3_ELB_LOGS_BUCKET_NAME: the name of the S3 bucket where to store the ELB logs.

### DC/OS
* DCOS_CONTROLLER_INSTALL_DEMOS: `true` - install demo(s) (default), `false` - skip demo installation.

#### Demos
* DCOS_CONTROLLER_INSTALL_DEMO_SD_AND_LB_NGINX: `true` - install demo setting up Nginx as described in [Service discovery and load balancing with DCOS and marathon-lb: Part 1](https://mesosphere.com/blog/2015/12/04/dcos-marathon-lb/).
* DCOS_CONTROLLER_INSTALL_TUTUM_HELLO_WORLD: `true` - install [Tutum hello-world](https://github.com/tutumcloud/hello-world) demo.

## Run a Docker container on Marathon

To run a Docker container on Marathon:

*docker.json*
```
{
  "id": "tutum-hello-world",
  "cpus": 0.2,
  "mem": 20.0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "tutum/hello-world",
      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 80, "hostPort": 0, "servicePort": 0, "protocol": "tcp" }
      ]
    }
  }
}
```
Post the container defintion to Marathon: `curl -X POST localhost:8080/v2/apps -d @Desktop\docker.json -H "Content-type: application/json"`.
