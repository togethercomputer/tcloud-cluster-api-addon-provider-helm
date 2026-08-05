# Together CAAPH release

Together's fork adds an opt-in `orphan-on-delete` handoff used by TCCO when a
`HelmChartProxy` repository changes. TCCO atomically sets the new repository
and this target-scoped annotation:

```yaml
helmchartproxy.addons.cluster.x-k8s.io/orphan-on-repository-change: "oci://new/repository"
```

When the value exactly matches the HCP's new `repoURL`, CAAPH adds this exact
annotation to each old HRP immediately before deleting it:

```yaml
helmreleaseproxy.addons.cluster.x-k8s.io/orphan-on-delete: "true"
```

Other immutable changes, ordinary deletion, and stale target annotations keep
the upstream uninstall behavior.

## One-time repository setup

The fork repository must be named
`togethercomputer/tcloud-cluster-api-addon-provider-helm`. The `tcloud-` prefix
is required by the current production GitHub OIDC trust policy.

Create the ECR repository from the accompanying `tcloud-infra` change before
publishing the first release.

## Publish

Start from a commit that passed the normal CAAPH tests, then create and push a
Together prerelease tag based on the upstream version:

```shell
git tag -a v0.3.1-together.1 -m "CAAPH v0.3.1 with safe release handoff"
git push origin v0.3.1-together.1
```

The tag workflow publishes:

- `651706779278.dkr.ecr.us-west-2.amazonaws.com/cluster-api-helm-controller:v0.3.1-together.1`
- GitHub release assets `addon-components.yaml` and `metadata.yaml`

The release must finish before enabling the matching custom `AddonProvider` in
`tcloud-infra`.
