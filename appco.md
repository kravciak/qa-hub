# AppCo pre-release

- [Gitlab project](https://gitlab.suse.de/orchid/suse-products-recipes/suse-security)
- [IBS project](https://build.suse.de/project/show/Devel:Jasmine:Charts) - subprojects are created, 1 per Merge Request

## Versioning (-alpha,-rc)
We are using an alpha version to build the containers.
We will not move MRs out of draft and merge until we have the stable version released by the community project and we pull the source-code for that stable version.

When there is a new version we:
 - create a MR in GitLab and then CI is triggered
 - it builds the Helm chart in an ephemeral IBS project and the built OCI artifact is pushed to the internal OCI registry, in an ephemeral repository
 - Both the IBS project and the OCI repository live as long as the MR in GitLab is open
 - naming always follow the same convention. To avoid collisions, we add the MR id in the paths, in principle, that's the only parameter
 - we build and push the containers in the same way

## Steps:

### Parameters:
- version: 0.8.0
- helm chart: mr-25
- containers: mr-33

### Create cluster with access to registry.suse.de (IBS)

- modified [cluster config](https://github.com/kubewarden/kubewarden-end-to-end-tests/blob/main/config/k3d-config-registry.suse.de.yaml) for registry.suse.de
- with `insecure_skip_verify: true` or SUSE ca


### Prepare namespace with pull secret
```bash
# Generate auth token https://apps.rancher.io/settings/access-tokens
set -x APPCO_ID <ID>
set -x APPCO_PW <PW>
kubectl create namespace kubewarden
kubectl create secret docker-registry application-collection -n kubewarden \
  --docker-server=dp.apps.rancher.io --docker-username=$APPCO_ID --docker-password=$APPCO_PW
```

### Install

You can set parameters from MRs manually. This example attempts to parse them from open MRs.
<!-- ver=1.0.0-3.1 -->
<!-- # tag=1.37.0 # 1.37.0-1.3 # 1? 1.37.0? -->
```bash
# Product name from MR
title='SUSE Security Admission Controller'

# Build oci url from chart MR id (30)
mrc=$(glab mr list -R https://gitlab.suse.de/orchid/suse-products-recipes/suse-security/charts --search "$title" -F json --jq '.[0].iid')
mr_chart=oci://registry.suse.de/devel/jasmine/charts/suse-security/mr-$mrc/charts/suse-security-admission-controller

# Build registry url from image MR id (38)
mri=$(glab mr list -R https://gitlab.suse.de/orchid/suse-products-recipes/suse-security/rpms-containers --search "$title" -F json --jq '.[0].iid')
mr_reg=registry.suse.de/devel/jasmine/containers/suse-security/mr-$mri

# Get product version (1.37.0)
mr_tag=$(glab mr list -R https://gitlab.suse.de/orchid/suse-products-recipes/suse-security/charts --search "$title" -F json --jq '.[0].title' | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+')
```

```bash
# Login required for released versions
# helm registry login dp.apps.rancher.io -u $APPCO_ID -p $APPCO_PW

# Admission Controller
helm install ssac $mr_chart --wait --namespace kubewarden -f - <<EOF
# Use auth secret
global:
  imagePullSecrets:
    - application-collection

# Admission Controller values
recommendedPolicies:
  enabled: true
auditScanner:
  policyReporter: true

---
# Use container images from MR (internal OCI registry)
image:
  registry: $mr_reg
  tag: $mr_tag
auditScanner:
  image:
    registry: $mr_reg
    tag: $mr_tag
policyServer:
  image:
    registry: $mr_reg
    tag: $mr_tag
EOF
```
