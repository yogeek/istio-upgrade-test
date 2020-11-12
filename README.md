# istio-upgrade-test

Test istio upgrade 1.5.4 ---> 1.6.13


# Install 1.5.4

envsubst < config_templates/custom_default_profile.yml | ./istio-1.5.4/bin/istioctl manifest apply -f -
envsubst < config_templates/ingressgateway-internal.yml | ./istio-1.5.4/bin/istioctl manifest generate -f - | kubectl apply -f -
