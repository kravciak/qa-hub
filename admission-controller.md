## Admission Controller

Running AC tests (CNCF, without UI):
```bash
# === Run tests on released AC ===
# Install AC & run bats e2e
make install VERSION=1.36 basic-tests.bats

# === Run tests on PR ===
# Checkout PR and pass local directory as parameter to makefile
gh pr checkout https://github.com/kubewarden/adm-controller/pull/1802
ln -s ../adm-controller/charts charts-devel
make install CHARTS_LOCATION=./charts-devel LATEST=1 basic-tests.bats
```
