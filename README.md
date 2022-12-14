# Deploy CP4BA - [ODM](https://www.ibm.com/products/operational-decision-manager?utm_content=SRCWW&p1=Search&p4=43700064670126812&p5=p&gclid=CjwKCAjwqJSaBhBUEiwAg5W9p_oekz-evt442nYbrYWGB6ouLhun87jWI4P7lbKOUoPAyx_5H5rSJhoCx7UQAvD_BwE&gclsrc=aw.ds)
## Prerequisites 
- Install `cloudctl`
    ```bash
    curl -L https://github.com/IBM/cloud-pak-cli/releases/latest/download/cloudctl-darwin-amd64.tar.gz -o cloudctl-darwin-amd64.tar.gz || curl -L https://github.com/IBM/cloud-pak-cli/releases/latest/download/cloudctl-darwin-amd64.tar.gz -o cloudctl-darwin-amd64.tar.gz
    ```
    ```bash
    tar -zxvf cloudctl-darwin-amd64.tar.gz || sudo -zxvf cloudctl-darwin-amd64.tar.gz
    ```
    ```bash
    mv cloudctl-darwin-amd64 $HOME/bin/cloudctl || sudo mv cloudctl-darwin-amd64 $HOME/bin/cloudctl 
    ```
### Infrastructure - Kustomization.yaml
1. Edit the Infrastructure layer `${GITOPS_PROFILE}/1-infra/kustomization.yaml` and un-comment the following:
    ```yaml
        - argocd/consolenotification.yaml
        - argocd/namespace-ibm-common-services.yaml
        - argocd/namespace-sealed-secrets.yaml
        - argocd/namespace-tools.yaml
        - argocd/namespace-db2.yaml
        - argocd/namespace-cp4ba.yaml
        - argocd/namespace-kube-system.yaml
        - argocd/namespace-openldap.yaml
        - argocd/serviceaccounts-ibm-common-services.yaml
        - argocd/serviceaccounts-tools.yaml
        - argocd/serviceaccounts-db2.yaml
        - argocd/serviceaccounts-cp4ba.yaml
        - argocd/serviceaccounts-norootsquash.yaml
    ```
1. Set Storage Classes 
    ```bash
    export CP_FILE_STORAGE=ibmc-file-gold-gid
    export CP_BLOCK_STORAGE=ibmc-block-gold
    ``` 
1. Adding your [Your Entitlement Key](https://myibm.ibm.com/products-services/containerlibrar)

    ```bash
    export IBM_ENTITLEMENT_KEY=<your entitlement key>
    ```
1. Create IBM Entitlment Key secret in `db2`, `kube-system` & `cp4ba` namespaces 
    ```bash
    oc create secret docker-registry ibm-entitlement-key -n db2 \
    --docker-server=cp.icr.io \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

    ```
    - Add the IBM entitlment key to namespace `kube-system`

    ```bash
    oc create secret docker-registry cpregistrysecret -n kube-system \
    --docker-server=cp.icr.io/cp/cpd \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

    ```
    ```bash
    oc create secret docker-registry ibm-entitlement-key -n cp4ba \
    --docker-server=cp.icr.io \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

    ```
    ```bash
    oc create secret docker-registry ibm-entitlement-key -n ibm-common-services \
    --docker-server=cp.icr.io \
    --docker-username=cp \
    --docker-password=${IBM_ENTITLEMENT_KEY} 
    ```
1. Create the catalog source
- You will need to get the value for `CASE_VERSION` by looking up the [CASE version of latest Db2 Operator](https://www.ibm.com/docs/en/db2/11.5?topic=deployments-db2-red-hat-openshift#concept_bq1_v4r_hlb__case-version).
    ```bash
    export CASE_REPO_PATH=https://github.com/IBM/cloud-pak/raw/master/repo/case
    export CASE_NAME=ibm-db2uoperator
    export CASE_VERSION=4.5.0
    export OFFLINEDIR=~/offline/db2/${CASE_VERSION}
    ```
- If you haven't yet created the directory, do so now:
    ```bash
    mkdir -p ${OFFLINEDIR}
    ```
- Download the `CASE` file:
    ```bash
    cloudctl case save --repo ${CASE_REPO_PATH} --case ${CASE_NAME} --version ${CASE_VERSION} --outputdir ${OFFLINEDIR}
    ```
- Extract the `CASE` bundle:
    ```bash
    cd ${OFFLINEDIR}
    tar -xvzf ${CASE_NAME}-${CASE_VERSION}.tgz
    ```
- Install the `Db2` catalog:
    ```bash
    cloudctl case launch --case ${CASE_NAME} --namespace openshift-marketplace --inventory db2uOperatorSetup --action installCatalog --tolerance 1
    ```
- Install the Operator:
    ```bash
    cloudctl case launch --case ${CASE_NAME} --namespace db2 --inventory db2uOperatorSetup --action installOperatorNative --tolerance 1
    ```
### Services - Kustomization.yaml

1. Edit the Services layer `${GITOPS_PROFILE}/2-services/kustomization.yaml` uncomment the following:
    ```yaml
    - argocd/operators/ibm-db2u-operator.yaml
    - argocd/operators/ibm-catalogs.yaml
    - argocd/instances/sealed-secrets.yaml
    - argocd/operators/ibm-foundations.yaml
    - argocd/instances/ibm-foundational-services-instance.yaml
    - argocd/operators/ibm-automation-foundation-core-operator.yaml
    ```

- Create operator group `db2-opeartorgroup.yaml`
- Give the service account privileged security content constraints (SCC) by running the following command:
    ```bash
    oc adm policy add-scc-to-user privileged system:serviceaccount:kube-system:norootsquash
    ```
- Deploy db2 operator `db2-sub.yaml`
- SCC for db2 `db2-scc.yaml`
- Validate by running this command:
    ```bash
    oc get db2ucluster db2ucluster-cp4ba -n tools
    ```
    or you can use the `watch` command to run the command every 2 secounds:
    ```bash
    watch "oc get db2ucluster -n tools"
    ```
- When the state field says Ready your instance is ready for use. [The Cloud Pak for Data documentation](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=db2-administering) has some handy information on `Administering Db2` should you need create users, tables, and other such stuff, like you will for Cloud Pak for Business Automation.

> Note! You can find the password for your instance in the `c-db2ucluster-cp4ba-instancepassword` secret in the `tools` namespace. You provided the password in the custom resource you used to create the instance, but it's easier to find it in this secret than it is to go and find the custom resource.

### Deploy LDAP


1. Edit the Services layer `${GITOPS_PROFILE}/2-services/kustomization.yaml` uncomment the following:
    ```yaml
    - argocd/instances/openldap.yaml
    ```
1. The following will provide Ldap url:
    ```bash
    oc get route -n openldap | grep openldap-admin | awk '{print "https://"$2}'
    ```



- Deploy Demonset `notrootsquash.yaml`
- Deploy the actual db `db2ucluster-instance.yaml` this should take around 15 min
    > Note!
    > Validate by doing the following:
        ```bash
        oc get db2ucluster db2ucluster-cp4ba -n db2
        ``` 
### Configuring Ldap
- New namesapce `namespace-openldap.yaml`
- Apply CRB `openldap-anyuid.yaml`
- we need to script the helm / k8's job for `helm`. 

### Deploy CP4A:
    ```bash
    oc create secret docker-registry cpregistrysecret -n cp4ba \
    --docker-server=cp.icr.io/cp/cpd \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 
    ```
1. Edit the Services layer `${GITOPS_PROFILE}/2-services/kustomization.yaml` uncomment the following:
    ```yaml
    - argocd/operators/ibm-cp4a-operator.yaml   
    ```
### Deploy ODM:
1. Edit the Services layer `${GITOPS_PROFILE}/2-services/kustomization.yaml` uncomment the following:
    ```yaml
    - argocd/instances/ibm-cp4ba-odm.yaml
    ```
- Validate the secrets:
    ```bash
    oc get secret -n cp4ba --sort-by='.metadata.creationTimestamp'
    ```
- Verify the sataus:
    ```bash
    oc get icp4acluster icp4adeploy -n cp4ba -o jsonpath="{.status}{'\n'}" | jq
    ```
### Post Deployment Steps:
- When the cloud pak is successfully installed you should be able to login as the default admin user. In this case, the default admin user is admin and the password can be found in the ibm-iam-bindinfo-platform-auth-idp-credentials secret in the cp4ba namespace. The URLs can be found in the icp4adploy-cp4ba-access-info configmap in the cp4ba namespace. Look for the section called odm-access-info
### Add user permissions
- You will need to grant your users various access roles, depending on their needs. You manage permissions using the Administration -> Access control page in the Cloud pak dashboard.

- Click on the hamburger menu on the top left corner of the dashboard; expand the Administration section and click on Access control.

- Click on the User Groups tab, then click on New user group.

- Name the group odmadmins, and click Next.

- Click Identity provider groups, then type cpadmins in the search field. It should come back with one result: cn=cpadmins,ou=Groups,dc=cp. Select it and click Next. Click all of the roles:

- Administrator
- Automation Administrator
- Automation Analyst
- Automation Developer
- Automation Operator
- ODM Administrator
- ODM Business User
- ODM Runtime administrator
- ODM Runtime user
- User
- Click Next, then click Create.