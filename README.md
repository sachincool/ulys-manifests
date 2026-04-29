# ulys-manifests

GitOps source of truth for the [ulys](https://github.com/sachincool/ulys-prod)
product. Argo CD reconciles cluster state from `main` of this repo;
nothing in here is applied by hand.

## Layout

```
argocd-applications/   App-of-Apps root + per-env children (dev, staging, prod)
platform/{base,dev,staging,prod}/
                       cluster-scoped: Linkerd, Flagger, ESO, OTel collector,
                       default-deny NetworkPolicy
apps/{base,dev,staging,prod}/
                       api/worker: Deployment, Service, Canary, ExternalSecret,
                       NetworkPolicy, Linkerd Server + AuthorizationPolicy
```

`base/` holds the canonical manifests; each env overlay is a thin
kustomization patching project IDs, hostnames, and image digests. CI's
`ci-app.yml` (in the source repo) opens digest-bump PRs against
`apps/<env>/kustomization.yaml`. Promotion between envs is a separate
manual workflow that copies a signed digest from one env's overlay to
the next.

## Sync policies

| env | sync | gate |
|---|---|---|
| dev | automated, prune+selfHeal | Flagger SLO check (auto-rollback on failure) |
| staging | automated, prune+selfHeal | Flagger SLO check |
| prod | manual | human approver in GitHub Environment "prod" + Flagger SLO check |
