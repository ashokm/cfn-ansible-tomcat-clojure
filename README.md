# cfn-ansible-tomcat-clojure

### Description

Uses a CloudFormation template to provision infrastructure and resources in AWS and performs a deployment of a Clojure based collector into the AWS environment.

Highly available architecture; utilising auto scaling groups and multiple availability zones.

### Technology stack:

#### AWS CloudFormation

CloudFormation is used to provision all the infrastructure resources in our AWS cloud environment.

#### Ansible

Ansible is used to configure the provisioned EC2 instances and:
1. Install Apache (latest)
2. Install JDK 8
3. Install Tomcat 8
4. Deploy the WAR file

### Launch

The CloudFormation template can be launched via the AWS web console or the AWS CLI.

The following example creates a new CloudFormation stack using the AWS CLI and the [cfn-template.json](cfn-template.json) file in this repository.

**Tip:** Use [direnv](https://direnv.net/) to setup your environment variables.

A `.envrc` file within your project directory can contain your AWS environment variables for the AWS CLI and allows for project-specific environment variables without cluttering your home profiles.

Example `.envrc` file:
```
export AWS_DEFAULT_REGION=eu-west-1
export AWS_ACCESS_KEY_ID=ABCDEFGHIJKLMNOPQRST
export AWS_SECRET_ACCESS_KEY=CKjzh7SHDwsxT2B93zL954wVMhHjaSnMtn6FNGt4
```

You can create the stack using the following example (replacing the parameter values with your own):

```
aws cloudformation create-stack --stack-name whatever-stack \
--template-body file://./cfn-template.json \
--parameters \
ParameterKey=VpcId,ParameterValue="vpc-93d63df7" \
ParameterKey=Subnets,ParameterValue="subnet-f955849d\\,subnet-7308f105" \
ParameterKey=OperatorEMail,ParameterValue="whatever@example.com" \
ParameterKey=KeyName,ParameterValue="whatever-keypair"
```

Full list of parameters:

| Parameter Name  | Description                                                       | Default                               |
| --------------- | ----------------------------------------------------------------- | ------------------------------------- |
| VpcId           | VpcId of your existing Virtual Private Cloud (VPC)                |                                       |
| Subnets         | The list of SubnetIds in your Virtual Private Cloud (VPC)         |                                       |
| InstanceType    | WebServer EC2 instance type                                       | m1.small                              |
| OperatorEMail   | EMail address to notify if there are any scaling operations       |                                       |
| KeyName         | The EC2 Key Pair to allow SSH access to the instances             |                                       |
| SSHLocation     | The IP address range that can be used to SSH to the EC2 instances | 0.0.0.0/0                             |
| InstanceCount   | Number of EC2 instances to launch                                 | 1                                     |
| HealthCheckPath | The destination path for health checks                            | /clojure-collector-1.1.0-standalone/i |

To see the status of the CloudFormation stack, use the describe-stacks command.

```
aws cloudformation describe-stacks --stack-name whatever-stack
```
To delete the CloudFormation stack, use the delete-stack command.
```
aws cloudformation delete-stack --stack-name whatever-stack
```

Note that the `UserData` property within the template will pull down this GitHub repo and execute the Ansible run.

### Verify
Providing all of the above runs successfully and the ALB Health Checks are in a `healthy` state, you should be able to access [http://endpoint:8080/clojure-collector-1.1.0-standalone/i](http://endpoint:8080/clojure-collector-1.1.0-standalone/i)

### Ideas for improvement

* Immutable AWS deployments
  * Build our own EC2 images (AWS AMIs) using [Packer](https://www.packer.io/) which contains the versioned application artifact:
    * Packer `template.json` stored in a VCS (Version Control System)
    * VCS update to trigger a CI build to clone the repo and run a Packer build to create an AMI (containing the versioned application artifact)
    * CI system to trigger CloudFormation job (to build or update AWS infrastructure)
    * Update ASG (Auto Scaling group) with new AMI
 * Update the Ansible playbook to follow [Best Practices](https://docs.ansible.com/ansible/2.5/user_guide/playbooks_best_practices.html) and allow the ability for the playbook to run on various OS distributions etc 

### References

* [AWS CloudFormation » User Guide » Sample Templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html)
* [Ansible Documentation](https://docs.ansible.com/)

### License and Authors

* Author: Ashok Manji
