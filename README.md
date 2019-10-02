# mig-hook-runner testing

## Scenario 1
Rolebinding to the mig-hook-runner serviceaccount access to multiple namespaces

1. `cd scenario-1-project-access
1. `oc create sa -n openshift-migration mig-hook-runner`
1. `oc new-project test`
1. `oc run -n test --image=ubi8-minimal sleep -- sleep infinity`
1. `oc create configmap -n openshift-migration mig-hook-runner-playbook-yml --from-file=mig-hook-runner-playbook.yml`
1. `oc create -f mig-hook-runner-job.yml`

At this point you should see the job container fail because the access is forbidden. Among the output:
```
{\"kind\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"pods is forbidden: User \\\"system:serviceaccount:openshift-migration:mig-hook-runner\\\" cannot list resource \\\"pods\\\" in API group \\\"\\\" in the namespace \\\"test\\\"\",\"reason\":\"Forbidden\"...
```

To fix this add the rolebinding in the test namespace to allow the openshift-migration:mig-hook-runner serviceaccount access:
1. `oc create -f mig-hook-runner-rolebinding.yml`
1. `oc delete job mig-hook-runner`
1. `oc create -f mig-hook-runner-job.yml`i

This time around you should see the job successfully complete and the pod details in the ansible debug message.

Note, while we will need to add a Rolebinding to give the hook runner pod access we won't want to restore it on the new cluster.

## Scenario 2
