# Releasing

Releases are cut from `main` and published as GitHub Releases.

## Preparing a new release

1. Pin the OpenTelemetry release and commit in [`versions.env`](versions.env).
2. Add changelog fragments for the upstream synchronization and all
   LoongSuite-specific changes.
3. Run the
   [Prepare release](https://github.com/alibaba/loongsuite-semantic-conventions-genai/actions/workflows/prepare-release.yml)
   workflow with the version pinned in `versions.env`.
4. Review and merge the pull request created by the workflow. If non-chore
   changes are merged while the release pull request is open, close it and
   prepare the release again.

The release pull request must contain:

- the OpenTelemetry schema copied from the pinned commit into
  `schemas/{version}`;
- the rendered changelog entry;
- generated documentation matching the YAML model.

## Publishing the release

1. Create a
   [new GitHub Release](https://github.com/alibaba/loongsuite-semantic-conventions-genai/releases/new).
2. Set the title and tag to `v{version}`.
3. Target the merge commit of the release preparation pull request.
4. Copy the new changelog section into the release notes and include the
   pinned OpenTelemetry tag and commit.
5. Verify the tag, generated schema, and release notes, then publish.
