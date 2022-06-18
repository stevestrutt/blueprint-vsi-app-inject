# Blueprints example to demonstrate 

The blueprint demonstrates deploying a full application stack composed from four Terraform configs

Following resources are deployed:
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
The vsi-app-blueprint.yaml definition file accepts the following inputs:


| Name | Type | Value | Description |
|------|------|------|----------------|
| region | string | us-east | Resource deployment region |
| resource_group_name | string | blueprint-rg-us | Resource group to create |
| provision_rg | string | true | Create RG - true. Use existing RG - false |
| cos_instance_name | string | vsi-app-cos-us | Name for COS instance |
| logdna_name | string | vsi-app-logdna-us | Name of LogDNA instance |
| vsi_name | string | bp-vsi-app-us | Name of VSIs |
| api_key | string | sensitive | User APIKey entered as env-var |
| ssh_public_key | string | sensitive | User SSH public key entered as env-var. The SSH key is not used, but a valid key must be specified to create the VSI |  

## Blueprints Outputs
The complex-blueprint.yaml definition creates the following outputs:

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


Sensitive input values like API keys and SSH keys are not defined in the input file as this potentially could lead to a security exposure when the  file is saved in the git repo. To pass sensitive values to Schematics, they are specified as environment variables when the environment is defined using the `ibcmcloud schematics environment create` command. To specify sensitive values, the input key is defined in the config file, but with a null value. This will cause the Schematics CLI to look for an env-var with the key name prefixed with `SCHEMATICS_ENV_`. 

The environment variable name is constructed by prefixing the config input key with `SCHEMATICS_ENV_` to create the full name.  

- api_key becomes SCHEMATICS_ENV_api_key
- ssh_public_key becomes SCHEMATICS_ENV_ssh_public_key

Export the config values as environment variables:
```
export SCHEMATICS_ENV_api_key='xxxxxxxxxxxxxxxxx'
export SCHEMATICS_ENV_ssh_public_key='rsa yyyyyyyyyyy'
```

The SSH key can be sourced using:
```
export SCHEMATICS_ENV_ssh_public_key=`cat ~/.ssh/id_rsa.pub`
```

Use an existing API Key or generate a new one at https://cloud.ibm.com/iam/apikeys


## Prerequisites
1. Install the Schematics CLI plugin by follow the instructions in the [documentation](https://cloud.ibm.com/docs/schematics?topic=schematics-setup-cli)  
2. Configure [IAM access permissions](https://cloud.ibm.com/docs/schematics?topic=schematics-access) for the Schematics Blueprints service. 
3. Set Schematics Target Region
The target (manage from) Schematics region for the Blueprint instance is determined by the IBM Cloud CLI target region. The region can be set with the `ibmcloud target` command.


## Usage 
**TEMPORARY USE OF payload_vsi-app.json file from /test folder for CLI testing**

**copy payload_vsiapp.json to CLI execution folder**  

Choose a Resource Group to associate the Blueprint with for Access Control. Available Resource Groups can be confirmed from the [Console Resource Groups](https://cloud.ibm.com/account/resource-groups) page.  

Depending on your account the RG in the payload_complex.json file may need changing to the name of an existing RG. Note some accounts have 'Default', others 'default'.  


```
$ ibmcloud target -r <region>

$ ibmcloud schematics blueprint create --file payload_vsiapp.json
```

CLI flag support due 27th June 2022
```
$ ibmcloud schematics blueprint create 
-name=VSI-APP
-resource_group=Default
-bp_git_url https://github.ibm.com/schematics-solution/blueprint-example-modules/vsi-app-blueprint.yaml
-input_git_url https://github.ibm.com/schematics-solution/blueprint-example-modules/vsi-app-input.yaml
```

```
$ ibmcloud schematics blueprint install -id blueprint_id

$ ibmcloud schematics blueprint jobs -id blueprint_id

$ ibmcloud schematics blueprint get -id blueprint_id

$ ibmcloud schematics blueprint destroy -id blueprint_id

$ ibmcloud schematics blueprint delete -id blueprint_id
```


