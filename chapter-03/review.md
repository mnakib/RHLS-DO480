
========== Deploying and Managing Policies with RHACM

===== RHACM for Kubernetes Governance Overview


===== RHACM Policy Overview



===== RHACM Policy Controllers Overview



===== DEMO - Guided Exercise

Create a policy in all the clusters to verify the expiration of the certificates in the openshift-console and openshift-ingress namespaces. Then, you will analyze the status of the policy, apply a fix, and verify the final status after the remediation.

0. Prepare the lab

$ lab start policy-governance

$ oc login -u admin -p redhat \
  https://api.ocp4.example.com:6443

$ oc new-project policy-governance


1. Log in to the RHACM web console and create the certificate policy

$ oc get routes -n open-cluster-management

$ firefox https://multicloud-console.apps.ocp4.example.com. &

    htpasswd_provider > admin | redhat

2. Create the certificate policy for both for openshift-console and openshift-ingress namespace

  Governance > Create policy
    
    - Name: policy-certificatepolicy
    - Namespace: policy-governance
    - Specifications: CertificatePolicy - Certificate management expiration
    - Remediation: Inform

    Do not create the policy yet. Edit the YAML code as follows:

      ...
      kind: CertificatePolicy
      metadata:
        name: policy-certificatepolicy-cert-expiration
      spec:
        namespaceSelector:
          include:
            - default
            - openshift-console
            - openshift-ingress
          exclude:
            - kube-*
      ...

    Click Create


    Review the status of the policy-certificatepolicy policy 
  
      Governance > policy-certificatepolicy > Clusters

        The status of the policy is Not compliant for the cluster named managed-cluster



3. Renew the ingress controller wildcard certificate of the managed-cluster.


$ cd ~/DO480/labs/policy-governance/

$ oc login -u admin -p redhat \
  https://api.ocp4-mng.example.com:6443

Run the renew_wildcard.sh script, that uses an Ansible Playbook to create a new wildcard certificate with an expiration date set to 3650 days from now.

$ ./renew_wildcard.sh

$ cd ~


4. Review the status of the policy-certificatepolicy policy

  Governance > policy-certificatepolicy

    The policy status is now Compliant for both clusters.


0.0 Finish the lab

$ lab finish policy-governance






========== Deploying and Configuring the Compliance Operator for Multiple Clusters Using RHACM

===== Definition of Compliance


===== Guided Exercise

Deploy the compliance operator on all the clusters in the Asia-Pacific (APAC) location to verify conformance to the Essential 8 (E8) standard. .

0. Prepare the lab

$ lab start policy-compliance

$ oc login -u admin -p redhat \
  https://api.ocp4.example.com:6443

$ oc new-project policy-compliance



1. Install the compliance operator

$ oc get routes -n open-cluster-management

$ firefox https://multicloud-console.apps.ocp4.example.com. &

    htpasswd_provider > admin | redhat

Create the compliance operator policy with the following parameters:

  - Name: policy-complianceoperator
  - Namespace: policy-compliance
  - Specifications: ComplianceOperator - Install the Compliance operator
  - Clustor selector: location: "APAC"
  - Remediation: Inform

  Do not create the policy yet. Edit the YAML code as follows:
      
      ...
      kind: Subscription
      metadata:
        name: compliance-operator
        namespace: openshift-compliance
      spec:
        installPlanApproval: Automatic
        name: compliance-operator
        source: do480-catalog
      ...
  
  Click Create.

  The source change is required only for the DO480 lab environment because the lab environment uses an offline operator catalog named do480-catalog.

  Review the status of the policy-certificatepolicy policy 

    Governance > policy-complianceoperator > Clusters

      The status of the policy is Not compliant for the cluster named managed-cluster.
      The Not compliant policy status indicates that the compliance operator is not installed on the managed-cluster.

  Enfore the policy

    Governance > policy-complianceoperator > Enforce


2. Verify the compliance operator installation

  Click the search icon:

    kind: subscription
    name: compliance-operator

  The search result shows the compliance-operator subscription present in the managed cluster.

  The APAC location has one cluster, named managed-cluster.



3. Clone the do480-policy-collection repository.

The do480-policy-collection repository has policy examples for Open Cluster Management.


$ git clone \
  https://github.com/RedHatTraining/do480-policy-collection.git


$ cd do480-policy-collection


4. Deploy the Essential 8 (E8) scan policy to the APAC location.

  Edit the policy-compliance-operator-e8-scan.yaml YAML file to change the following parameters:

  remediationAction: enforce
  key: location
  values: APAC

  The policy-collection repository has both stable and community policy examples. This course uses the E8 scan policy that is available under the stable → CM-Configuration-Management directory.


  $ cd \
  stable/CM-Configuration-Management

  $ vim \
policy-compliance-operator-e8-scan.yaml

    ...
    policy.open-cluster-management.io/categories: CM Configuration Management
    policy.open-cluster-management.io/controls: CM-6 Configuration Settings
spec:
  remediationAction: enforce
    ...
    matchExpressions:
      - {key: location, operator: In, values: ["APAC"]}
    ...


  $ oc login -u admin -p redhat \
  https://api.ocp4.example.com:6443

  $ oc project policy-compliance

  $ oc create -f \
policy-compliance-operator-e8-scan.yaml


  $ cd ~



5. Check the compliance operator E8 scan results.

  Governance > policy-e8-scan > Clusters

    The clusters tab shows three resources: compliance-e8-scan, compliance-suite-e8, and compliance-suite-e8-result. Compliance-suite-e8-result shows Not compliant.


  Click View details and review the details. On the details page, you can see the name of non-compliant objects.

    https://redpiranha.net/news/essential-8-strategies-mitigate-cyber-security-incidents#:~:text=The%20Essential%208%20%28E8%29%20is%20a%20prioritised%20subset,reduce%20the%20risk%20of%20a%20cybersecurity%20incident%20occurring.


  To list the compliance check results, click search and type:

    kind:configurationpolicy

  Click compliance-suite-e8-results and check the status. It shows a list of NonCompliant objects.


0.0 Finish the lab

  $ lab finish policy-compliance


