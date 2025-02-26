# NAIS Alert Config
Alert config for SSBs NAIS applications

## What is this?

This reusable workflow deploys a Kubernetes resource called `AlertManagerConfig`
in your team's namespace. The [AlertManagerConfig](https://docs.openshift.com/container-platform/4.11/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1.html) resource defines configuration
for all alerts in that namespace, including information as which Slack channel
the alert sends to, and how the Slack message is composed. Contributions are welcome!

In order to deploy this configuration, the deploy variables `team` (NAIS team) and `cluster` (Kubernetes cluster) needs to be set. The alerts are then
sent to the Slack channel `#<team>-nais-alerts-<cluster>`. 

Furthermore, since the `AlertManagerConfig` resource needs to be supplied to the 
NAIS deploy action in the workflow caller, this repo needs to be checked out and fetched
from the parent workflow. A sample:

```yaml
name: Deploy alerts
run-name: Deploy alerts for Pseudo Service to test and prod

on:
  push:
    branches:
      - master
    paths:
      - ".nais/alerts.yaml"
      - ".github/workflows/alert-deploy.yml"
  workflow_dispatch:
permissions:
  id-token: write
env:
  TEAM: some-team

jobs:
  test-deploy:
    name: Deploy alerts to test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - uses: actions/checkout@v4
        name: Retrieve AlertManager configuration
        with:
          repository: "statisticsnorway/nais-alert-config"
          path: "ext_alertconfig"
          sparse-checkout: |
            alertconfig.yaml
          sparse-checkout-cone-mode: false

      - name: Deploy to test
        uses: nais/deploy/actions/deploy@v2
        env:
          CLUSTER: test
          RESOURCE: .nais/alerts.yaml,ext_alertconfig/alertconfig.yaml
          VAR: cluster=test,team=${{ env.TEAM }}
          DEPLOY_SERVER: deploy.ssb.cloud.nais.io:443

  prod-deploy:
    name: Deploy alerts to prod
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - uses: actions/checkout@v4
        name: Retrieve AlertManager configuration
        with:
          repository: "statisticsnorway/nais-alert-config"
          path: "ext_alertconfig"
          sparse-checkout: |
            alertconfig.yaml
          sparse-checkout-cone-mode: false

      - name: Deploy to prod
        uses: nais/deploy/actions/deploy@v2
        env:
          CLUSTER: prod
          RESOURCE: .nais/alerts.yaml,ext_alertconfig/alertconfig.yaml
          VAR: cluster=prod,team=${{ env.TEAM }}
          DEPLOY_SERVER: deploy.ssb.cloud.nais.io:443
```