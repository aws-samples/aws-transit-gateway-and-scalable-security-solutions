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




## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

