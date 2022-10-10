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

### last steps:

```bash
    catalogsource.operators.coreos.com/ibm-cp4a-operator-catalog created
    catalogsource.operators.coreos.com/ibm-cp-automation-foundation-catalog created
    catalogsource.operators.coreos.com/ibm-automation-foundation-core-catalog created
    catalogsource.operators.coreos.com/opencloud-operators created
    catalogsource.operators.coreos.com/bts-operator created
    catalogsource.operators.coreos.com/cloud-native-postgresql-catalog created
    IBM Operator Catalog source created!
    Waiting for CP4A Operator Catalog pod initialization
    Waiting for CP4A Operator Catalog pod initialization
    CP4BA Operator Catalog is running ibm-cp4a-operator-catalog-g2h62                                   1/1   Running     0     31s
    operatorgroup.operators.coreos.com/ibm-cp4a-operator-catalog-group created
    CP4BA Operator Group Created!
    subscription.operators.coreos.com/ibm-cp4a-operator-catalog-subscription created #NOT FOUND
```