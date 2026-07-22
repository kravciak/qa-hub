## Runtime Enforcer

Testing process:

- Currently testing them manually.
- Planning to use Shepherd for test case automation. https://github.com/rancher/suse-security-tests
- Currently contributing to E2E test cases with runtime-enforcer.  https://github.com/rancher-sandbox/runtime-enforcer/tree/main/test/e2e

Running E2E tests on existing cluster

`E2E_USE_EXISTING_CLUSTER=true E2E_SKIP_DEPENDENCIES= go test -v -timeout 20m ./test/e2e/`

if cert-manager and cert-manager-csi-driver is installed already you can set E2E_SKIP_DEPENDENCIES=true
