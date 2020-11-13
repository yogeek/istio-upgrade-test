# istio-upgrade-test

Test istio upgrade 1.5.4 ---> 1.6.13


# Starting point

I am starting from a 1.5.4 istio installation which has been done with istioctl with 2 IstioOperator yaml files (one based on defaut profile and the other on empty profile in order to add another gateway istio-ingressgateway-internal) :

```
envsubst < config_templates/custom_default_profile.yml | ./istio-1.5.4/bin/istioctl manifest apply -f -
envsubst < config_templates/ingressgateway-internal.yml | ./istio-1.5.4/bin/istioctl manifest generate -f - | kubectl apply -f -
```

# Goal : uprade to 1.6.x version with no downtime

Following [the canary upgrade guide](https://istio.io/latest/docs/setup/upgrade/#canary-upgrades), I tried these steps :

```
# https://istio.io/latest/news/releases/1.6.x/announcing-1.6/upgrade-notes/

# Remove the ValidatingWebhookConfiguration Custom Resource (CR)
kubectl delete ValidatingWebhookConfiguration istio-galley


./istio-1.6.13/bin/istioctl install -n istio-system -f <(envsubst < ./config_templates/custom_default_profile.yml) -f <(envsubst < ./config_templates/ingressgateway-internal.yml) --set revision=1-6-13

# After running the command, we have 2 control plane deployments and services:
kubectl get pods -n istio-system -l app=istiod
kubectl get svc -n istio-system -l app=istiod

# And 2 sidecar injector configurations 
kubectl get mutatingwebhookconfigurations

# Operator is now installed !
kubectl get istiooperators.install.istio.io 
NAME                     REVISION   AGE
installed-state-1-6-13   1-6-13     2m8s
```

**BUT** only 1 ingress is updated :(
Plus this is my additionnal ingresgateway-internal and not the one by default...

# kubectl rollout restart deployment istio-ingressgateway
```

# How to uninstall a canary revision ?
 
It will be possible in 1.7 with the experimental feature :  `istioctl x uninstall --revision=1-6-13`
But in < 1.7...? 

I tried this :
```
./istio-1.6.13/bin/istioctl manifest generate -f <(envsubst < ./config_templates/custom_default_profile.yml) -f <(envsubst < ./config_templates/ingressgateway-internal.yml) --set revision=1-6-13 | kubectl delete -f -
```

But it removed also components from the old installatin (CRDs, etc.) => old installation is broken :(



# Goal 2 : upgrade to 1.7.x version with no downtime
