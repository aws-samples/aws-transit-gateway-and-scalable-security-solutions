The repository consists of all the templates that are required and detailed instructions on securing ingress using scalable security solutions (eg: Palo Alto) and transit gateway.  

## STEP 1: Set up Firewall Template

•	Create a S3 bucket with root folder structure as below.

•	Modify the init-cfg.txt file with vm-auth-key and panorama-server information.

•	Upload the init-cfg.txt file to the /config folder on the S3 bucket.

•	Upload the AWS Lambda code Firewall template (panw-aws.zip) and Application template (ilb.zip) to the S3 bucket’s root folder as shown in above screenshot.

•	Firewall Template:

   -	Launch the stack using (firewall-new-vpc-v2.1.template) template in the AWS CloudFormation console in the AWS account where you launch firewalls.
   
   -	Ensure that you select at least two availability zones.
   
   -	Enter the VM-Series-Firewall AMI ID. You need to subscribe to the produce (PAYG/BYOL). For this demonstration we are using ‘ami-056149984080d92af’ 
      in us-west-2 region.
      
   -	Select the existing Key pair for the VMs from the drop down menu.
   
   -	Enter the CIDR to allow SSH into VMs. In this case it is 0.0.0.0/0.
   
   -	Choose “Yes” for Enable debug log.



## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

