# Creating a ROSA STS cluster with an unmanaged OIDC configuration ID

This Terraform example creates a ROSA STS cluster that uses an unmanaged OIDC configuration. For more information on OIDC configurations, see the [OpenID Connect Overview](https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/introduction_to_rosa/rosa-oidc-overview) in the Red Hat Customer Portal.

## Prerequisites

* You have an AWS account. Your locally configured AWS credentials are used to access to AWS API for validation purposes.
* You have installed the latest version ROSA CLI (`rosa`).
* You must have an offline OCM token. This token can be generated in the [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift/token).
* You have installed Terraform. See the [Hashicorp Terraform page](https://developer.hashicorp.com/terraform/downloads) for the latest version.
* You have created your [account-wide IAM Roles](https://github.com/terraform-redhat/terraform-provider-ocm/blob/a42779d6b6712f4dde358344f44b782e4dfcd120/examples/create_rosa_cluster/create_rosa_sts_cluster/classic_sts/account_roles/README.md). For more information, see [Account-wide IAM role and policy reference](https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/introduction_to_rosa/rosa-sts-about-iam-resources#rosa-sts-account-wide-roles-and-policies_rosa-sts-about-iam-resources) in the Red Hat Customer Portal.


## ROSA cluster creation

1. To run the `terraform apply` you need to set up some variables. This guide uses environmental variables. For more on Terraform variables, see [Managing Variables](https://developer.hashicorp.com/terraform/enterprise/workspaces/variables/managing-variables) in the Terraform documentation.

   > **NOTE**: If you exported these variables in your current command-line session when running the account-roles Terraform example, you do not need to export them again.

   Run the following commands to export your variables. Provide your values in lieu of the brackets. Note that any values declared in the `variables.tf` are the default values if you do not export a superseding value.
        
    1. Your account role prefix prepends to all of your created roles. This value should match the value you set when creating your account roles. This value cannot end with a hyphen (-).
        ```
        export TF_VAR_account_role_prefix=<account_role_prefix>
        ```
    2. The availability zones should be a zone within your AWS region, which you chose with the `TF_VAR_cloud_region` variable. This value must be enclosed by double quotes, a bracket pair, and single quotes.
        ```    
        export TF_VAR_availability_zones='["<aws_az_region"]' 
        ```
    3. This value sets the AWS region where your cluster launches.
        ```
        export TF_VAR_cloud_region=<aws_region_name>
        ```
    4.  This value is your cluster name. The cluster name cannot exceed fifteen (15) characters or the Terraform apply fails.  
        ```
        export TF_VAR_cluster_name=<cluster_name>
        ```
    5.  This value is your installer role's ARN. This role was created with the account roles terraform process. 
        ```
        export TF_VAR_installer_role_arn=<cluster_name>
        ```
    6.  Your Operator role prefix prepends all of your Operator roles and policies. You should use the same value as your account-role prefix for consistency, though it is not required.
        ```
        export TF_VAR_operator_role_prefix=<operator_role_prefix>
        ```
    7.  This variable should be your full [OCM offline token](https://console.redhat.com/openshift/token) that you generated in the prerequisites.  
        ```
        export TF_VAR_token=<ocm_offline_token>
        ```
    8.  This value should point to your OpenShift instance.  
        ```
        export TF_VAR_url=<ocm_url>
        ```
    9.  **Optional**: You can set the desired OpenShift version with this variable. The default is 4.13.
        ```    
        export TF_VAR_openshift_version=<choose_openshift_version>
        ```
    10.  **Optional**: If you want to set any specific AWS tags for your cluster, you can use this variable to declare those tags.   
         ```    
         export TF_VAR_tags=<aws_resource_tags> (Optional) 
         ```         
1. In your local copy of the `cluster_with_managed_oidc_config` folder, run the following command:
   ````
   terraform init
   ````
   Running this command accesses all the necessary provider information to apply your Terraform plan.
1. **Optional**: Run the `plan` command to ensure that your Terraform files build correctly without errors. This is not required to apply your Terraform plans.
   ````
   terraform plan -out rosa-cluster.tfplan
   ````
1. Run the apply command to create your ROSA cluster. 

   > **Note**: If you did not run the `plan` command, you can simply just `apply` without specifying a file.

    ````
    terraform apply <"rosa-cluster.tfplan">
    ````
1. The Terraform applies the plan and creates your cluster. You will see a prompt to confirm you want to create these resources. Enter `yes`, then the process will complete with your resources.

## Verification

1. In your command-line interface, run the following command to verify that the cluster was created:
    ````
    rosa list cluster
    ````
1. You see your roles when the command finishes. 
    ````
    ID                                NAME            STATE  TOPOLOGY
    24cg9rtij6pshqaqrgephn2rrfte1pd6  user-rosa-tf-2  ready  Classic (STS)
## Resource clean up

After you are done with the resources you created, you should not delete them manually, but instead, use the `destroy` command. Run the following to delete all of your created resources:
  
    terraform destroy

After the command is complete, your resources are deleted.

> **NOTE**: If you manually delete a resource, you create unresolvable issues within your environment.
