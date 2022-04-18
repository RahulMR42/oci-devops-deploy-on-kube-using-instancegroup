Deploy kubernetes using OCI Devops and Instances group

----------

Its a specific sample execution to cover 

- OCI Devops with Instance group deployment.
- While the instance is acted as a jump host towards a customer kubernetes cluster.


Target Audience : OCI Devops Experienced

Basic instructions 

- Create a dynamic group with below rules 

```
ALL {resource.type = 'devopsbuildpipeline', resource.compartment.id = 'ocid1.compartment.oc1..xx'}	
0
ALL {resource.type = 'devopsdeploypipeline', resource.compartment.id = 'ocid1.compartment.oc1..xx'}	
1
ALL {resource.type = 'devopsrepository', resource.compartment.id = 'ocid1.compartment.oc1..xx'}	
2
All {instance.compartment.id = 'ocid1.compartment.oc1..xx'}	
3
ALL {resource.type = 'devopsconnection', resource.compartment.id = 'ocid1.compartment.oc1..xx'}
```

- Create a policy with below statements 


```
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage repos in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage repos in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to read secret-family in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to read secret-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage devops-family in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to manage devops-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage generic-artifacts in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to manage generic-artifacts in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to use ons-topics in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to use ons-topics in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to use instance-agent-command-execution-family in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to use instance-agent-command-execution-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to read generic-artifacts in compartment <compartment_name>Allow dynamic-group mr-telesis-private-dg-devops-instances to read generic-artifacts in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to read all-artifacts in compartment mr-prod-compartment

```

- Create an OCI artifact registry - https://docs.oracle.com/en-us/iaas/Content/artifacts/home.htm

![](images/oci_artifact_repo.png)

- Create an OCI Container registry - https://docs.oracle.com/en-us/iaas/Content/Registry/home.htm


![](images/oci_container_repo.png)


- Create an OCI notiication topic - https://docs.oracle.com/en-us/iaas/Content/Notification/home.htm

![](images/oci_topic.png)


- Create a devops project - https://docs.oracle.com/en-us/iaas/Content/devops/using/create_project.htm#create_a_project 

![](images/oci_project.png)


- Create an OCI code repo - https://docs.oracle.com/en-us/iaas/Content/devops/using/create_repo.htm#create_repo 


![](images/oci_coderepo.png)

- Follow the on screen  instruction and push the whole files here to OCI code repo.


![](images/oci_coderepo_2.png)


- Create an OCI instance (Image must be Oracle Linux or Centos) - https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/launchinginstance.htm

- Use a cloud init inline script and elevate sudo access for the user.



```
#cloud-config
users:
  - default
  - name: ocarun
    sudo: ALL=(ALL) NOPASSWD:ALL
```

![](images/oci_instances_2.png)

![](images/oci_instances.png)


- Create a build pipeline - https://docs.oracle.com/en-us/iaas/Content/devops/using/create_buildpipeline.htm#create_buildpipeline 






- Create a deploy pipeline - https://docs.oracle.com/en-us/iaas/Content/devops/using/deployment_pipelines.htm 






