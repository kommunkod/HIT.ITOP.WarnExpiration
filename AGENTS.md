# Repository Instructions

## Versioning

Release this extension with a tag. The downstream GitLab pipeline updates this extension's version pin in `hit-itop-docker/composer.json`; do not manually edit that pin.

## Commits

Keep commit messages short and specific. Do not mention AI, agents, or AI-generated credits in commits.

## Releases

Follow the release instructions in `README.md`. Use `glab` to run the release pipeline without a version argument so it auto-increments the version, then approve its manual release job. Wait for the extension pipeline and its triggered `hit-itop-docker` pipeline to complete before proceeding.
