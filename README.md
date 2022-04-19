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
Allow dynamic-group mr-telesis-private-dg-devops-instances to read secret-family in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to read secret-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage devops-family in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage devops-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage generic-artifacts in compartment <compartment_name>
llow dynamic-group mr-telesis-private-dg-devops-instances to manage generic-artifacts in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to use ons-topics in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to use ons-topics in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to use instance-agent-command-execution-family in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to use instance-agent-command-execution-family in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to read generic-artifacts in compartment <compartment_name>
Allow dynamic-group mr-telesis-private-dg-devops-instances to read generic-artifacts in compartment mr-prod-compartment
Allow dynamic-group  to read all-artifacts in compartment mr-prod-compartment
Allow dynamic-group mr-telesis-private-dg-devops-instances to manage all-resources in compartment <compartment_name>

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

- We are using kubectl for the kubernetes deployment ,to do so you may install kubectl manually to the compute instance or add a step inside the deployment stage.

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ 

- Create a devops environment with the instance information - https://docs.oracle.com/en-us/iaas/Content/devops/using/environments.htm

![](images/oci_env.png)

- Create artifacts - https://docs.oracle.com/en-us/iaas/Content/devops/using/artifacts.htm

- Artifact for Container Image ,with type as Container Image (The image will be used to deploy on to the target kubernetes).

![](images/oci_artifact_1.png)

- Artifact for Kubernetes manifest (A yaml to use with kubectl)

![](images/oci_artifact_2.png)

- Artifact for instance group deployment (To orchestrate instance actions )

![](images/oci_artifact_2.png)

- Create a build pipeline - https://docs.oracle.com/en-us/iaas/Content/devops/using/create_buildpipeline.htm#create_buildpipeline 


- Add  a manged build stage. 

![](images/oci_build_1.png)

- Add a deliver artifact stage.Select all 3 artifacts and associate with the correct outputArtifact name from build_spec file.

![](images/oci_build_1.png)

- Create a deploy pipeline - https://docs.oracle.com/en-us/iaas/Content/devops/using/deployment_pipelines.htm 


![](images/oci_deploy_1.png)

- Add a deploy to instance group stage to the deploy pipeline

![](images/oci_deploy_1.png)

- Switch back to build pipeline and add a trigger deployment stage to connect build and deployment.

![](images/oci_deploy_1.png)


- Here we are using an OKE to simulate as an `external` kubernetes ,so these values to adjust at the build_spec.yaml

![](images/oci_build_spec.png)


```
variables:
    artifact_repo_ocid: <OCID of artifact repo>
    oke_oci_odi: <OCI of OKE>
    region: <OCI Region>
    oci_oke_endpoint_type: <Cluster API Endpoint type  - PUBLIC_ENDPOINT or PRIVATE_ENDPOINT>
```

- Upload the changes back to OCI Code repo and do a manual run.

- Once the build completes follow the deployment and finally validate the kubernetes deployments (Its a web app ,can be launched using the service loadbalancer IP)

- You may use cloud-shell or bastion hosts to verify the kubernetes deployments as well.


![](images/build_done.png)


Lets think of another scenario
-----

- In this case we are using a standard kubernetes and using token we will be creating a kubeconfig.

- Create a vault and store the kubeapi server name and port as well as access token to vault secret .

- Add the policy for build runner to access vault secrets 

```
Allow dynamic-group xxxx to read secret-family in compartment <compartment_name>
```

Example .

server=https://x.x.x.x:port

token="kube access token"

- Rename  build_spec_kube.yaml as build_spec.yaml

- Rename instance_deploymentspec_kube.yaml as instance_deploymentspec.yaml

- Re run the build pipeline and validate.





Read more 
----

- OCI Devops - https://docs.oracle.com/en-us/iaas/Content/devops/using/home.htm.
- OCI Reference architectures  -  https://docs.oracle.com/solutions/
- OCI Devops samples - https://github.com/oracle-devrel/oci-devops-examples 



Contributors 
===========

- Author : Rahul M R.
- Colloboroators : NA
- Last release : March 2022