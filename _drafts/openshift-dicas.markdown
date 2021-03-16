# Apagando PODs Evicted (de um namespace específico)

```bash
oc project idp
oc get pods | awk '{if ($3=="Evicted") print "oc delete pod " $2;}' | head
```

# Apagando PODs Evicted DE TODOS OS NAMESPACES

```bash
oc get pod --all-namespaces  | awk '{if ($4=="Evicted") print "oc delete pod " $2 " -n " $1;}' | sh
```

# Obter os valores em texto plano das secrets

```bash
oc get secret SECRET_NAME -o json |\
  jq -r '.data' |\
  head -n -1 |\
  tail -n +2 |\
  awk '{"echo "$2"| sed s/,// | base64 -d " | getline x; print $1, x}'
```
// TODO: revisar o comando acima
