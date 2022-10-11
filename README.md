- Deploying the namespace 
- add the IBM entitlment key in DB2 namespace 
    ```bash
    oc create secret docker-registry ibm-entitlement-key -n db2 \
    --docker-server=cp.icr.io \
    --docker-username=cp \
    --docker-password=<IBM_ENTITLEMENT_KEY_REPLACE> 

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
    --docker-password=<IBM_ENTITLEMENT_KEY_REPLACE> 

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
- Namespaces for cp4a and common-services with ibm-entitlment-key
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