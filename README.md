# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2026 The Linux Foundation

# Central Publish Action

Publish Maven artifacts to Maven Central via the [Central Portal REST API](https://central.sonatype.com/publishing).

## Features

- GPG signs all artifacts (`.jar`, `.pom`, `.module`)
- Creates compliant bundle ZIP for Central Portal upload
- Supports `AUTOMATIC` (auto-publish) and `USER_MANAGED` (validate-only) modes
- Polls deployment status until completion
- `dry-run` mode for local testing (creates bundle, skips upload)
- Generates `$GITHUB_STEP_SUMMARY` with results

## Usage

### Basic (validate only — safe for testing)

```yaml
- uses: askb/central-publish-action@main
  with:
    m2repo-path: m2repo
    central-username: ${{ secrets.CENTRAL_USERNAME }}
    central-token: ${{ secrets.CENTRAL_TOKEN }}
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY_B64 }}
    publishing-type: USER_MANAGED
```

### Production (auto-publish)

```yaml
- uses: askb/central-publish-action@main
  with:
    m2repo-path: m2repo
    central-username: ${{ secrets.CENTRAL_USERNAME }}
    central-token: ${{ secrets.CENTRAL_TOKEN }}
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY_B64 }}
    publishing-type: AUTOMATIC
```

### Dry run (no upload)

```yaml
- uses: askb/central-publish-action@main
  with:
    m2repo-path: m2repo
    central-username: ${{ secrets.CENTRAL_USERNAME }}
    central-token: ${{ secrets.CENTRAL_TOKEN }}
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY_B64 }}
    dry-run: "true"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `m2repo-path` | yes | `m2repo` | Path to local Maven repo directory |
| `central-username` | yes | — | Central Portal token username |
| `central-token` | yes | — | Central Portal token password |
| `gpg-private-key` | yes | — | GPG private key (base64-encoded armor) |
| `gpg-passphrase` | no | _(empty)_ | GPG key passphrase |
| `publishing-type` | no | `USER_MANAGED` | `AUTOMATIC` or `USER_MANAGED` |
| `dry-run` | no | `false` | Skip upload, create bundle only |
| `poll-timeout` | no | `600` | Max seconds to wait for validation |
| `poll-interval` | no | `15` | Seconds between status polls |

## Outputs

| Output | Description |
|--------|-------------|
| `deployment-id` | Central Portal deployment ID |
| `deployment-status` | Final status: `VALIDATED`, `PUBLISHED`, or `FAILED` |
| `bundle-path` | Path to the created bundle ZIP |

## Requirements

### Maven Central POM Compliance

Your POM must include:
- `<name>` — project name
- `<description>` — project description
- `<url>` — project URL
- `<licenses>` — at least one license
- `<developers>` — at least one developer
- `<scm>` — source control info

### Artifacts Required

For each module:
- `*.jar` — compiled artifact
- `*.pom` — POM file
- `*-sources.jar` — source code
- `*-javadoc.jar` — Javadoc

The action generates `.asc` (GPG signature) for each file automatically.

## How It Works

```
1. Import GPG key from base64 secret
2. Sign all .jar/.pom/.module files → .asc signatures
3. Create bundle.zip (Maven directory structure, excludes checksums)
4. POST bundle to Central Portal: /api/v1/publisher/upload
5. Poll /api/v1/publisher/status until VALIDATED/PUBLISHED/FAILED
```

## Testing

Use `publishing-type: USER_MANAGED` for safe testing:
- Artifacts are uploaded and validated
- NOT published to Maven Central
- Can be deleted from Portal UI
- Perfect for CI testing

## License

Apache-2.0
