# GitHub Action for running Verify.Cli

Perform snapshot testing with files in your GitHub Action workflows. If the received file doesn't match the verified file, then the action will return an error and it will run `diff` against the two files so you can see what the difference is.

Runs [`flcdrg/Verify.Cli`](https://github.com/flcdrg/Verify.Cli) in Docker and prints a unified diff when verification
fails. Use a Linux runner with Docker available, and check out your repository
before running the action. A failed verification fails the action.

## Simplest Use

Verify an existing file against `dist/tags.html.verified.html`. Only `file`
is required; the working and snapshot directories default to `dist`.

```yaml
steps:
  - uses: actions/checkout@v7
  - name: Verify tags
    uses: flcdrg/verify-cli-action@v1
    with:
      file: dist/tags.html
```

## Additional Inputs

Use workspace-relative paths and compare against
`verified/tags.html.verified.html`, scrubbing changing content before comparison.
In both examples, generate `dist/tags.html` before the verification step if it
is not checked into the repository.

```yaml
steps:
  - uses: actions/checkout@v7
  - name: Verify tags
    uses: flcdrg/verify-cli-action@v1
    with:
      file: dist/tags.html
      working-directory: .
      verified-directory: verified
      scrub-inline-pattern: |
        (?<prefix>")/_astro/[^"]+(?<suffix>")
        (?<prefix>title=")[^"]+(?<suffix>")
        (?<prefix>meta name="generator" content="Astro v)[\d\.]+(?<suffix>")
      scrub-inline-remove: ' data-image-component="true"'
      verbosity: detailed
```

## Inputs

| Input                  | Required | Default                 | Description                                                                                  |
| ---------------------- | -------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| `file`                 | Yes      |                         | File path relative to the explicit working directory, or the current directory when omitted. |
| `working-directory`    | No       | File's parent directory | Directory to mount at `/tmp` in the container.                                               |
| `verified-directory`   | No       | `.`                     | Snapshot directory relative to the working directory.                                        |
| `scrub-inline-pattern` | No       |                         | Regular expressions, one per line.                                                           |
| `scrub-inline-remove`  | No       |                         | Literal strings to remove, one per line.                                                     |
| `verbosity`            | No       | `detailed`              | Verify.CLI verbosity level.                                                                  |

Blank scrub lines are ignored; all other whitespace and backslashes are
preserved. Keep file and snapshot paths within the mounted working directory.
When `working-directory` is omitted, `file: dist/tags.html` mounts `dist`
and verifies `/tmp/tags.html`; the default snapshot directory is also `dist`.
Set `working-directory: .` to keep paths relative to the workspace, as above.
On failure, the action lists the snapshot directory and compares
`<filename>.verified.<extension>` with `<filename>.received.<extension>`.
No separate diff step is needed.
