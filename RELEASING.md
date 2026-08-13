# Releasing

This package publishes to the private Composer registry (`git-registry.hoglandet.se`) and, on release, triggers a new `hit-itop-docker` image build that pins in the new version.

## Cutting a release

1. Go to **CI/CD → Pipelines** in GitLab and click **Run pipeline** on the branch you want to release from (usually `main` or `develop`).
2. Optionally set a `VERSION` variable (e.g. `1.3.0`). If you leave it unset, the pipeline automatically bumps the **minor** version by one based on the latest existing tag (e.g. `1.2.4` → `1.3.0`).
3. Run the pipeline and let the manual `release` job run. It will:
   - Update `composer.json`'s `version` field to the resolved version.
   - Commit that change and create a git tag on the same commit.
   - Push both, which automatically triggers a new (tag) pipeline.

## What happens automatically after that

- The tag pipeline's `upload-tag` job publishes exactly that version to the registry.
- The `trigger-downstream-develop`/`trigger-downstream-main` jobs pass `PACKAGE_NAME=hoglandetsit/itop-warn-expiration` and `PACKAGE_VERSION=<the new version>` into the `hit-itop-docker` pipeline.
- `hit-itop-docker` pins **only this package** to the new version in its own `composer.json`/`composer.lock` (every other package is left untouched) and builds a new image.

## Checking what landed in a build

- The `hit-itop-docker` pipeline's `build-image` job log prints every package's pinned version and the build ID before building.
- The built image also ships a CycloneDX SBOM at `/opt/itop/sbom.json` (`docker exec <container> cat /opt/itop/sbom.json`), which includes the build ID.
- The image is tagged with that build ID (`hoglandet.azurecr.io/itop:<CI_PIPELINE_ID>`) in addition to the existing floating tags (`develop`/`latest`/`rc`/core version).

## One-off debug builds with different package versions

Run `hit-itop-docker`'s pipeline manually from the GitLab UI with a `PACKAGE_OVERRIDES` variable, e.g.:

```
PACKAGE_OVERRIDES=hoglandetsit/hit-itop-organization:1.2.3,hoglandetsit/hit-itop-person:2.0.0
```

This builds an image using those versions for that run only — it does **not** commit anything back to `hit-itop-docker`'s `composer.lock`.

## Deploying to staging/TST

Once `build-image` succeeds, the `deploy-staging` job in `hit-itop-docker`'s pipeline waits for a manual approval click before rolling out to the staging Kubernetes cluster (`hitoptst01.intern.hoglandet.se`).
