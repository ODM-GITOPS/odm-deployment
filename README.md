## Prerequisites 
### Install cloudctl
```bash
    curl -L https://github.com/IBM/cloud-pak-cli/releases/latest/download/cloudctl-darwin-amd64.tar.gz -o cloudctl-darwin-amd64.tar.gz || curl -L https://github.com/IBM/cloud-pak-cli/releases/latest/download/cloudctl-darwin-amd64.tar.gz -o cloudctl-darwin-amd64.tar.gz
```
```bash
    tar -zxvf cloudctl-darwin-amd64.tar.gz || sudo -zxvf cloudctl-darwin-amd64.tar.gz
```
```bash
    mv cloudctl-darwin-amd64 $HOME/bin/cloudctl || sudo mv cloudctl-darwin-amd64 $HOME/bin/cloudctl 
```
- Deploying the namespaces db2, kube-system, cp4ba
- Set Storage Classes 
```bash
export CP_FILE_STORAGE=ibmc-file-gold-gid
export CP_BLOCK_STORAGE=ibmc-block-gold
``` 
[Your Entitlement Key](https://myibm.ibm.com/products-services/containerlibrar)
```bash
export IBM_ENTITLEMENT_KEY=<your entitlement key>
```


- Create IBM Entitlment Key secret in `db2`, `kube-system` & `cp4ba` namespaces 
    ```bash
    oc create secret docker-registry ibm-entitlement-key -n <NAMESPACE> \
    --docker-server=cp.icr.io \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

    ```
- Add catalog source, `ibm-db2uoperator-catalog`
- Create operator group `db2-opeartorgroup.yaml`
- Deploy db2 operator `db2-sub.yaml`
- SCC for db2 `db2-scc.yaml`
- Add the IBM entitlment key to namespace `kube-system`

    ```bash
        oc create secret docker-registry cpregistrysecret -n kube-system \
    --docker-server=cp.icr.io/cp/cpd \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

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

### CP4A:
    ```bash
        oc create secret docker-registry cpregistrysecret -n cp4ba \
    --docker-server=cp.icr.io/cp/cpd \
    --docker-username=cp \
    --docker-password=$IBM_ENTITLEMENT_KEY 

    ```
- catalog_source
- ibm-common-services "ibm-catalogs"
- operator group
- common-services-operator-group
- sub
- ldap
### odms:
- ibm-ban-secret
- odm-db-secret
- oc get secret -n cp4ba --sort-by='.metadata.creationTimestamp'
- on icp4adeploy spec change the version to 21.0.1
- icp4adeploy instance