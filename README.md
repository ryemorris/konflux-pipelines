# Remote pipeline definitions for Konflux

When a component is onboarded in Konflux, two `PipelineRun` custom resource definitions are automatically created in the Git repository of the component:
- `${component.name}-pull-request.yaml`
- `${component.name}-push.yaml`

By default, these files will contain an inline pipeline definition which can be found under the `pipelineSpec` YAML key.

## How to use remote pipeline definitions

This repository offers an alternative to inline pipeline definitions based on [remote pipelines from Tekton](https://tekton.dev/docs/pipelines/resolution/).
To use a remote pipeline definition from this repository, you just need to replace `pipelineSpec` with `pipelineRef` in your `PipelineRun` custom resource definition, as showed below or in [konflux-pipelines#16](https://github.com/RedHatInsights/konflux-pipelines/pull/16):

```yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata: ... # Omitted for brevity
spec:
  params: ... # Omitted for brevity
  pipelineRef:
    params:
      - name: url
        value: https://github.com/RedHatInsights/konflux-pipelines
      - name: revision
        value: main
      - name: pathInRepo
        value: pipelines/docker-build.yaml
    resolver: git
  workspaces: ... # Omitted for brevity
```

## Benefits

- Remote pipeline definitions help minimize the efforts required to manage and review PRs generated by the `red-hat-konflux` bot, like [konflux-pipelines#12](https://github.com/RedHatInsights/konflux-pipelines/pull/12).
- Teams can leverage Git’s versioning to depend on specific pipeline definition branches.
- In case of issues, it’s very simple to roll back to a previous pipeline definition version, and apply it globally across multiple Konflux components.

## Drawback

- When an _inline_ pipeline definition is updated by the `red-hat-konflux` bot, the Konflux component will usually be immediately built and tested in Konflux.
When a _remote_ pipeline definition is updated, the Konflux component will _not_ be built or tested, as the `red-hat-konflux` bot will submit a PR in this repository and not in the Git repository of the component.
