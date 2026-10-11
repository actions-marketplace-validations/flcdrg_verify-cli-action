# Repository Guidance

## Scope

- Implement action behavior in [action.yml](action.yml); this is a composite Bash action, not a .NET application. Verify.CLI runs in an external Docker image.
- Read [README.md](README.md) for inputs, examples, snapshot naming, and directory defaults. Update it when the public input contract changes.
- Reuse [.github/workflows/test.yml](.github/workflows/test.yml) and the existing `samples/` fixtures; there is no application build or package-install step.

## Action And Test Changes

- Pass inputs through environment variables and quoted Bash arrays. Preserve whitespace and backslashes in scrub values, including leading spaces in removal strings.
- Resolve both host and container paths when changing directory handling. Keep source files and snapshots inside the mounted directory; for sibling data and snapshot directories, mount their common parent.
- Keep verification and failure-diff directory resolution consistent. Check workflow input names against the action metadata.
- Preserve intentional mismatch fixtures and tracked verified snapshots. Do not accept received snapshots automatically; `*.received.*` files are ignored.
- Expected-failure tests must inspect `steps.<id>.outcome` with an `always()` condition, not `success()` or the continued step's `conclusion`. A skipped step is not an expected failure.
- Do not count missing files, missing baselines, or Docker errors as successful mismatch tests. Check freshly generated received snapshots when strengthening these tests.

## Validation

- Run `actionlint .github/workflows/test.yml` when actionlint is available. YAML parsing alone does not validate GitHub Actions expressions or step outcomes.
- Syntax-check both embedded Bash scripts after action edits:

```sh
ruby -ryaml -ropen3 -e 'YAML.load_file("action.yml").fetch("runs").fetch("steps").each { |step| _, error, result = Open3.capture3("bash", "-n", stdin_data: step.fetch("run")); abort error unless result.success? }'
```

- Parse README YAML examples after editing them; ensure fenced examples use spaces rather than tabs and headings start at the left margin.
- Use the existing Linux/Docker workflow for runtime validation. Mocked Docker checks can verify arguments and exit propagation but do not prove Verify.CLI behavior; state that distinction in results.
- For a reported CI run, inspect that run's logs and commit rather than assuming current local files match it. Separate the primary failure from downstream skips and assertion failures.
