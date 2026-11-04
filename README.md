# CAPZ Aster

This repository is the CAPZ consumer configuration for [Aster](https://github.com/willie-yao/aster). It discovers public CAPZ Prow results and deploys the dashboard to Kubernetes through Flux.

Configuration changes follow the normal pull request, CI, merge, and Flux reconciliation path. Runtime Secrets are never stored in Git. AI analysis uses a bounded CAPZ configuration. Authenticated analysis chat, reviewed write actions, and Agent Sandbox Fix previews are enabled. GitHub writes require explicit human confirmation.

The default public endpoint is Azure Front Door:

	https://capz-aster-99e3d335-c8fva4csbpcuh2e8.b01.azurefd.net

## GitHub read authentication

The engine images are pinned to Aster commit `2779b0f70ea4910d9fccbeb41f8259611ed954db`. Worker and server continue to use `GITHUB_READ_TOKEN` from `capz-aster-model`. This upgrade preserves the existing authentication and does not renew that token automatically.

GitHub App authentication is available in the engine but is not enabled here. Before switching to it:

1. Create or reuse a GitHub App that allows installation on both required organizations. Grant **Contents: Read-only**, **Metadata: Read-only** (included automatically), and **Pull requests: Read-only** for CAPZ triage and escalation. Read authentication does not require Issues write permission.
2. Obtain repository administrator or organization owner approval and install the App on **`kubernetes-sigs/cluster-api-provider-azure`** and **`kubernetes/test-infra`**. A consumer-only installation is insufficient, including for these public repositories. Test-infra access is required for job discovery.
3. Generate an App private key and provision the separate `capz-aster-github-app` Secret in namespace `capz-aster` through the Secret owner's workflow. It must contain `ASTER_APP_ID` and `ASTER_APP_PRIVATE_KEY`. For a manual installation, read both values from protected files:

   ```sh
   kubectl --context aster-capz-prow-dashboard --namespace capz-aster create secret generic capz-aster-github-app \
     --from-file=ASTER_APP_ID=/secure/path/app-id \
     --from-file=ASTER_APP_PRIVATE_KEY=/secure/path/private-key.pem
   ```

After these prerequisites are verified, replace the static token reference with required App Secret references in both `fetcher.extraEnv` and `server.extraEnv`, then review and merge that configuration through the normal Flux workflow. The published chart `0.12.0-rc.3` supports those environment entries but predates the newer `githubApp.existingSecret` helper.

In App mode, missing credentials prevent the workloads from starting; a missing installation, denied permission, or renewal failure fails the affected read without falling back to a PAT or anonymous access. App credentials select App reads even though pull request commenting is disabled. The model-provider Secrets and `BOT_TOKEN` write identity keep their existing roles.

Installation tokens renew in memory without pod restarts. When rotating the App private key, update the Secret, restart both worker and server to reload their environment, verify the rollout, then revoke the old key. Updating an environment-backed Secret alone does not refresh running processes. See [Aster's GitHub read authentication guide](https://github.com/willie-yao/aster/blob/2779b0f70ea4910d9fccbeb41f8259611ed954db/docs/github-read-auth.md).
