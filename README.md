# parcel-mixed-final

Disposable Advanced mixed role scaffold

| Attribute | Value |
| --- | --- |
| System | `system:default/parcel-dispatch` |
| Implementation profile | `quarkus-camel-openapi` |
| Approved build profile | `quarkus-jvm` |
| Build service | Tekton/OpenShift Pipelines |
| Schema Registry API | `https://apicurio.apps.cluster-ggvsm.dyn.redhatworkshops.io/apis/registry/v3` |

This repository owns implementation source and developer tooling. Runtime and build desired state
live under this Component in the parent System repository. Tekton clones this public repository
without a Git write credential.

## After Creation

1. Review and merge the Component pull request in the parent System repository.
2. Wait for the Component environment, runtime, and initial build to become healthy.
3. Confirm that the `git-<full-commit-sha>` image exists before creating a human release.

## Release and Promotion

```text
push to main
    -> build git-<full-sha>

push vX.Y.Z tag
    -> attach the human release tag to the existing image digest

Promote Component in Developer Hub
    -> update the desired release in the next environment
```

Create a release by tagging a commit that has already built successfully:

```bash
git tag -a v1.7.3 <commit> -m "Release v1.7.3"
git push origin v1.7.3
```

Creating the Git tag does not rebuild the Component. Release materialization resolves the
commit's existing `git-<full-sha>` image and adds the human tag to that digest. It is asynchronous;
wait for the tag to appear in Quay.

Creating a release does not promote an environment. After materialization succeeds, run
**Promote Component** in Developer Hub. That workflow opens a pull request to update
`components/parcel-mixed-final/releases/<environment>.yaml` in the parent System
repository. Merging it selects the release for the next environment and starts target-local image
promotion. To roll back, select an older release through the same workflow.

The shared release guard permits an absent human tag or one already resolving to the same digest;
it rejects attempts to move an existing tag to a different artifact.

## Local Development

Use the checked-in implementation tooling so local and pipeline builds share project configuration.
Java profiles provide the Maven wrapper (`./mvnw test`); the Node.js profile uses reproducible npm
commands (`npm ci && npm test`). The `.devfile.yaml` provides the matching editor and workspace
entrypoint.

## Runtime Configuration

Runtime configuration is owned in the parent System repository:

| Path | Purpose |
| --- | --- |
| `components/parcel-mixed-final/values.yaml` | Build and shared Component intent |
| `components/parcel-mixed-final/environments/<environment>.yaml` | Replicas, Route, health checks, resources, environment variables, and Secrets |
| `components/parcel-mixed-final/releases/<environment>.yaml` | Selected image tag only |

Keep runtime configuration out of release files so promotion and rollback change only the
artifact selection.

## API Dependencies

The generated build configuration records each selected contract and downloads it from the Schema
Registry. The selection behavior is the same across implementation profiles:

| Selection | Build behavior |
| --- | --- |
| `latest` | Follows the latest available Registry publication |
| Human release tag | Downloads the matching immutable Registry version |
| Exact Git SHA | Downloads the matching immutable commit publication |

`latest` can change a generated model without a Component source change. Prefer a release or SHA
for repeatable builds. Missing Registry artifacts fail the build without falling back to Git or
catalog content.
Java builds download contracts during Maven `initialize` and write them under
`target/generated-resources/openapi`.

## Consumed APIs
This Component does not currently consume an API.

Cross-System contract consumption is supported in v1. Runtime endpoint resolution is owned by
environment configuration and may override the generated default shown above. CF-IDP does not
automatically discover runtime endpoints from catalog or contract metadata.
