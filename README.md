## Steps to get alerts from OCP cluster

From each version of ocp cluster run this (much readable version of script (Thanks to Sunil) will be updated soon but for now this works :) )

```bash
# switch to openshift-monitoring namespace
oc project openshift-monitoring;

# create token
oc create token cluster-monitoring-operator > cmo-token;

# get thanos querier route
THANOS_QUERIER_ROUTE=$(oc get routes -n openshift-monitoring thanos-querier -o 'jsonpath={.status.ingress[0].host}');

curl -sk -H "Authorization: Bearer $(cat cmo-token)" \
https://$THANOS_QUERIER_ROUTE/api/v1/rules | \
jq -cr  '.data.groups | map(select(.file | contains("openshift-monitoring") or contains("openshift-user-workload-monitoring"))) | map(.rules) | flatten | map(select(.type=="alerting")) | sort_by(.name)|map([.name,.labels.severity,.duration,.query,.annotations.summary, .annotations.description,"----"]) | map(.|join("\n") ) |.[]' > monitoring-rules-$(oc version -o json | jq -cr .openshiftVersion).txt
```
