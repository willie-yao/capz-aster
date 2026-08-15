# CAPZ Aster

This repository is the CAPZ consumer configuration for [Aster](https://github.com/willie-yao/aster). It discovers public CAPZ Prow results and deploys the dashboard to Kubernetes through Flux.

Configuration changes follow the normal pull request, CI, merge, and Flux reconciliation path. Runtime Secrets are never stored in Git. AI analysis, chat, write actions, and Fix generation are initially disabled.

The default public endpoint is Azure Front Door:

	https://capz-aster-99e3d335-c8fva4csbpcuh2e8.b01.azurefd.net
