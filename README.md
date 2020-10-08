The repository consists of all the templates that are required and detailed instructions on securing ingress using scalable security solutions (eg: Palo Alto) and transit gateway.  

## STEP 1: Set up Firewall Template

* Create a S3 bucket with root folder structure as below.

![s3_bucket_root_folder](/images/s3_bucket_root_folder.png)


*	Create a S3 bucket with root folder structure as below.

*	Modify the [init-cfg.txt](https://github.com/PaloAltoNetworks/aws-elb-autoscaling/blob/master/Version-2.1/panorama_sample_config/init-cfg.txt) file with vm-auth-key and panorama-server information.

*	Upload the init-cfg.txt file to the **/config** folder on the S3 bucket.

*	Upload the AWS Lambda code Firewall template ([panw-aws.zip](https://github.com/PaloAltoNetworks/aws-elb-autoscaling/blob/master/Version-2.1/firewall/panw-aws.zip)) and Application template ([ilb.zip](https://github.com/PaloAltoNetworks/aws-elb-autoscaling/blob/master/Version-2.1/apps/ilb.zip)) to the S3 bucket’s root folder as shown in above screenshot.

*	Firewall Template:
      * Launch the stack using ([firewall-new-vpc-v2.1.template](https://github.com/PaloAltoNetworks/aws-elb-autoscaling/blob/master/Version-2.1/firewall/firewall-new-vpc-v2.1.template)) template in the [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.Walkthrough.html) console in the AWS account where you launch firewalls.
      * Ensure that you select at least two availability zones.
      * Enter the VM-Series-Firewall [AMI ID](https://docs.paloaltonetworks.com/vm-series/9-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/deploy-the-vm-series-firewall-on-aws/obtain-the-ami/get-amazon-machine-image-ids.html). You need to subscribe to the produce (PAYG/BYOL). For this demonstration we are using ‘ami-056149984080d92af’ in us-west-2 region.
      * Select the existing Key pair for the VMs from the drop down menu.
      * Enter the CIDR to allow SSH into VMs. In this case it is 0.0.0.0/0
      * Choose “**Yes**” for **Enable debug log**.

      ![picture1](/images/picture1.png)
      
      * Specify the name of the S3 bucket created earlier for bootstrapping firewall.
      * Specify the S3 bucket containing panw-aws.zip file.
      * The API-Key for firewall is configured with a default username and password. The “pandemo/demopassword” can be changed from Panorama.
      * Enter the previously generated Panorama API key.
      * Enter the Admin username for Panorama. (sample config username is ‘pandemo’)
      * Enter the name of the Load balancer.
      * Click next and launch the stack.
      * Once complete, note the following parameters:
         * AutoScaling group launched instances in each availability zone.
         * Network load balancer SQS queue name
         * Elastic load balancer DNS name

## STEP 2: Set up TGW:
* Create a stack using tgw.yaml in AWS Cloudformation console in firewall Account.
* Enter the transit gateway name.
* Enter the Organization ID that enables sharing of the Transit gateway with the spoke accounts.

_Note: The above template creates transit gateway with two routing domains - DMZ VPC routing domain and the Spoke VPC routing domain._


## STEP 3: Enable connectivity between spoke account/VPCs and DMZ VPC
a. In DMZ VPC
* Create a stack using tgw-attachment.yaml in AWS Cloudformation console in the firewall account VPC.
* Provide the transit gateway ID created in step 2.
* Provide the trusted subnets of your DMZ VPC.
* Provide the Routing table ID associated to the trusted subnets.
* Provide the Spoke VPC CIDR to which you connect to (It is best to summarize the spoke VPCs CIDR here). This is to enable routing between DMZ VPC and Spoke VPCs. 
* Click next and create.

_Note: Routing tables of trusted subnets need to be updated with Spoke VPC CIDR as destination and Transit gateway as target_

b. In Spoke VPC
* Create a stack again using tgw-attachment.yaml in AWS Cloudformation console in Spoke account/VPC.
* Provide the Transit gateway ID created in step 2.
* Provide the internal subnets of your spoke VPC.
* Provide the DMZ VPC CIDR to which the spoke VPC need to connect to. This is to enable routing between a spoke VPC and DMZ VPC.
* Provide the Routing table ID associated to the spoke vpc subnets.
* Click next and create.
* Repeat the above steps in all Spoke VPCs

c. In DMZ account/VPC
* Create another stack using tgw-propagations.yaml in AWS Cloudformation console in DMZ account/VPC.
* Provide the ID for the DMZ VPC.
* Provide the IDs for the Spoke VPCs.
* Provide the names for your Spoke VPCs in the same order as IDs for VPCs.
* Provide the TGW ID.
* Click next and create.

_Note: Launching the above stack will create a custom resource and a lambda function which requires to be updated each time a new spoke is connected to the transit gateway._

## STEP 4: Set up Application Template:
* Launch the stack using panw-aws-alb-existing-vpc-v2.1-updated.template template in the [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.Walkthrough.html) console in the spoke VPC account.
* Provide the template name. Here we are using Application-Template
* Select the existing Application Spoke VPC from the drop-down list for the VPCID.
* Select the number of availability zones and AZ from the list.
* Provide the CIDRs for trust subnets from application VPC.
* For ILBSubnets, select the Subnets from Application Spoke VPC that need to be associated with Internal Load Balancer (Enter Subnet IDs).
* Give the name for Internal Application Load Balancer.
* Under the Lambda section, enter the S3 bucket name containing Lambda zip file (‘ilb.zip’).
* Enter the SQS Queue URL copied from output of the Firewall stack.
* Select the key value pair from the list.
* Enter the CIDR to allow SSH into VMs. In this case it is 0.0.0.0/0

![picture2](/images/picture2.png)

![picture3](/images/picture3.png)

* Under the VPC Connectivity section, enter the total number of AZs for the DMZ VPC.
* Enter the trust subnet CIDRs from DMZ VPC.
* If Cross account:
  * Select False for ‘Same Account Deployment’.
  * Enter the Cross-Account Role ARN.
  * Enter the AWS Account ID for DMZ VPC.

![picture4_v2](/images/picture4.png)

  
## STEP 5: Update the Listener Rules
* Create a stack using ALBRule.yaml in AWS Cloudformation console in the DMZ account VPC.
* Enter the DMZ VPC ID.
* Enter the DNS name for the application.
* Enter the Public LoadBanlancer Listener ARN which was launched with the Firewall template.
* Enter the port number unique for the application. This should be same as VM NAT policy configuration rule added by the Application template.
* Provide the name for your new Target Group.
* Once the template launch completes, navigate to the [AWS AutoScaling service](https://console.aws.amazon.com/ec2autoscaling/home) console.
* Click on the AutoScaling group launched by the Firewall Template.
* In the ‘Details’ section, look for ‘Load balancing’ and then Click ‘Edit’.
* Select the newly created Target group under ‘Choose a target group for your load balancer’.
* Click ‘Update’.
* This should add the existing VM instances behind this target group.
* Modify the Health check path if required. Default is ‘/’.


Reference:
* For information regarding Fortinet device visit [here](https://aws.amazon.com/quickstart/architecture/fortinet-fortigate/)
* For information regarding Cisco device visit [here](https://github.com/CiscoDevNet/cisco-ftdv/tree/master/autoscale/aws/NGFWv6.6.0)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

