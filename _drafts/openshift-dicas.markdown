# Apagando PODs Evicted (de um namespace específico)

```bash
oc project idp
oc get pods | awk '{if ($3=="Evicted") print "oc delete pod " $2;}' | head
```

# Apagando PODs Evicted DE TODOS OS NAMESPACES

```bash
oc get pod --all-namespaces  | awk '{if ($4=="Evicted") print "oc delete pod " $2 " -n " $1;}' | sh
```
