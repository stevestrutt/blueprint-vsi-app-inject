# Blueprint multi-module application stack example

This blueprint demonstrates how infrastructure environments can be composed and scaled by linking Terraform configs. In this example four Terraform configs are linked to deploy a working Nginix application stack on VSIs within a VPC. After deployment the Nginix website can be accessed via the application URL returned by Schematics.   

The following resources are created:
- Resource Group
- COS instance and bucket
- LogDNA
- VPC
- VSI
- Nginx app 

## Blueprint definition - vsi-app-blueprint.yaml

The blueprint links four Terraform configs as Workspaces. 
- bp-vsi-resource-group
- bp-vsi-cos-storage
- bp-vsi-Observability
- bp-vsi-vpc-app

TF configs are sourced from https://github.com/Cloud-Schematics/blueprint-example-modules
```
Blueprint file: vsi-app-blueprint.yaml
├── bp-vsi-resource-group
|    └── source: github.com/Cloud-Schematics/blueprint-example-modules/IBM-ResourceGroup
├── bp-vsi-cos-storage
|    └── source: github.com/Cloud-Schematics/blueprint-example-modules/IBM-Storage
├── bp-vsi-Observability
|    └── source: github.com/Cloud-Schematics/blueprint-example-modules/IBM-Observability
└── bp-vsi-vpc-app
     └── source: github.com/Cloud-Schematics/blueprint-example-modules/IBM-VPC-App
```

The blueprint demonstrates deploying a full application stack composed from 4 modules. The included TF examples should not be considered best practice and are only included to demonstrate the layered approach to deploying applications and data passing. LogDNA and COS instance/buckets are not essential to this example and are only included as representative examples. 



### Blueprint definition inputs 
The vsi-app-blueprint.yaml definition accepts the following inputs:


| Name | Type | Value | Description |
|------|------|------|----------------|
| region | string | null | Resource deployment region |
| resource_group_name | string | null | Resource group to create |
| provision_rg | string | null | Create RG - true. Use existing RG - false |
| cos_instance_name | string | null | Name for COS instance |
| logdna_name | string | null | Name of LogDNA instance |
| vsi_name | string | null | Name of VSIs |
| ssh_public_key | string | sensitive | User SSH public key entered as env-var. The SSH key is not used, but a valid key must be specified to create the VSI |  

## Blueprints Outputs
The vsi-app-blueprint.yaml definition creates the following outputs:

| Name | Type | Value | Description |
|------|------|------|----------------|
| schematics-app-url | string |  | URL of sample NGINX application |

### Input file - vsi-app-input.yaml
The input file defines the variable values for all the required Blueprint definition inputs. Review the file contents to observe the difference in formating for the yaml scalar and block scalar formats. 

| Name | Type | Value | Description |
|------|------|------|----------------|
| region | string | us-east | Sample var for resource deployment region |
| resource_group_name | string | blueprint-vsi-us | Name of resource group to create |
| provision_rg | string | null | Create RG - true. Use existing RG - false |
| vsi_name | string | bp-vsi-app-us | Name of VSI instance |
| logdna_name | string | vsi-app-logdna-us |  |
| cos_instance_name | string | vsi-app-cos-us  | Name for COS instance |
| ssh_public_key: | string | null  | SSH Key  |


Sensitive input values like API keys and SSH keys should not be defined in the input file as this potentially could lead to a security exposure when the  file is saved in the git repo. To pass sensitive values to Schematics, they are passed as dynamic input variables when the Blueprint is created. Shell variable substution can be used to insert environment variables as input to Blueprint commands. 

The SSH key can be sourced and saved as the environment variable `user_ssh_key` using:
```
export user_ssh_key=`cat ~/.ssh/id_rsa.pub`
```



## Prerequisites
1. Install the Schematics CLI plugin by follow the instructions in the [documentation](https://cloud.ibm.com/docs/schematics?topic=schematics-setup-cli)  
2. Configure [IAM access permissions](https://cloud.ibm.com/docs/schematics?topic=schematics-access) for the Schematics Blueprints service. 
3. Configure [IAM access permissions](https://cloud.ibm.com/docs/vpc?topic=vpc-managing-user-permissions-for-vpc-resources) for IaaS VPC resources
4. Set Schematics Target Region
The target (manage from) Schematics region for the Blueprint instance is determined by the IBM Cloud CLI target region. The region can be set with the `ibmcloud target` command.
5. Export `user_ssh_key` for VSI creation 



The following parameters are used for the `blueprint create` configuration. 
- Name of the blueprint: `Blueprint_vsi_app`
- Schematics management resource group: `Default`
- Blueprint URL: `https://github.com/Cloud-Schematics/blueprint-vsi-app-example`
- Blueprint file: `vsi-app-blueprint.yaml`
- Blueprint Git branch `main`
- Input file URL: `https://github.com/Cloud-Schematics/blueprint-vsi-app-example`
- Input file: `vsi-app-input.yaml` 
- Input file Git branch `main`
- Inputs:
    - provisiong_rg=false
    - resource_group_name=Default
    - ssh_public_key=$user_ssh_key


Example commands
```sh
$ ibmcloud target -r <region>

$ export user_ssh_key=`cat ~/.ssh/id_rsa.pub`

$ ibmcloud schematics blueprint create -name Blueprint_Complex -resource-group Default -bp-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -bp-git-branch main -bp-git-file vsi-app-blueprint.yaml -input-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -input-git-branch main -input-git-file vsi-app-input.yaml -inputs provision_rg=false,resource_group_name=Default,ssh_public_key=$user_ssh_key


$ ibmcloud schematics blueprint install -id blueprint_id

$ ibmcloud schematics blueprint job list -id blueprint_id

$ ibmcloud schematics blueprint get -id blueprint_id -profile outputs

$ ibmcloud schematics blueprint destroy -id blueprint_id

$ ibmcloud schematics blueprint delete -id blueprint_id
```

## Next Steps

Looking for more samples? Check out the [{{site.data.keyword.bplong_notm}} GitHub repository](https://github.com/orgs/Cloud-Schematics/repositories/?q=topic:blueprint). 

Check the example Readme files for further Blueprint customisation and usage scenarios for each sample. 

### Customise blueprint-vsi-app-example to use alternative resource group
In a shared account the user may not have access permissions to create resources in the Default resource group. The Blueprint can be customised to use an alternative group that the user does have access permissions to. Replace `<USER_RESOURCE_GROUP>` with the alternative resource group name.  


```
$ ibmcloud schematics blueprint create -name Blueprint_Complex -resource-group <USER_RESOURCE_GROUP> -bp-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -bp-git-branch main -bp-git-file vsi-app-blueprint.yaml -input-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -input-git-branch main -input-git-file vsi-app-input.yaml -inputs provision_rg=false,resource_group_name=<USER_RESOURCE_GROUP>,ssh_public_key=$user_ssh_key
```




### Customise blueprint-vsi-app-example to create a resource group
In Pay-Go or Subscription accounts, or where the user has administrative permissions to create resource groups, the Blueprint can be customised to create a user specified resource group. The user must be an account owner or have been granted  [Account Management, editor or administrator permissions](https://cloud.ibm.com/docs/account?topic=account-account-services&interface=ui#account-management-actions-roles) to create resource groups. 

With the required IAM permissions, a user specified resource group can be created by changing the folllowing input parameters on the create command:
- `provision_rg=true` 
- `resource_group_name=<USER_RESOURCE_GROUP>`

```
$ ibmcloud schematics blueprint create -name Blueprint_Complex -resource-group Default -bp-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -bp-git-branch main -bp-git-file vsi-app-blueprint.yaml -input-git-url https://github.com/Cloud-Schematics/blueprint-vsi-app-example -input-git-branch main -input-git-file vsi-app-input.yaml -inputs provision_rg=true,resource_group_name=<USER_RESOURCE_GROUP>,ssh_public_key=$user_ssh_key
```

This will create the resource group `<USER_RESOURCE_GROUP>` and the COS instance will be created in this group. 

Refer to the [Schematics FAQ](https://cloud.ibm.com/docs/schematics?topic=schematics-blueprints-faq&interface=ui#faqs-bp-basic-example) documentation for diagnosing and resolving the typical configuration errors with this example and their resolution.  

