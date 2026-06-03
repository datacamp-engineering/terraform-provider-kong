## Agent guidance specific to this repo

This is DataCamp's fork of [kevholditch/terraform-provider-kong](https://github.com/kevholditch/terraform-provider-kong), a Terraform provider written in Go that manages Kong API gateway resources (services, routes, plugins, consumers, upstreams, targets, certificates, SNIs) via the Kong Admin API.

### Fork divergence

The critical fork-specific change is in `go.mod`: the upstream `github.com/kevholditch/gokong` client library is replaced with DataCamp's own fork `github.com/datacamp-engineering/gokong`. All Kong API calls flow through that client; changes to Kong-facing behaviour should be made there, not in this provider, unless they are Terraform schema / lifecycle concerns.

### Resource layout

All provider logic lives under `kong/`. Each resource follows the pattern `kong/resource_kong_<name>.go` with a corresponding `kong/resource_kong_<name>_test.go`. The provider is registered in `kong/provider.go`; `main.go` is only the plugin entrypoint. There are no data sources wired in (the `DataSourcesMap` block is commented out).

### Building

```sh
go build .                        # compile only
make build                        # goimports check + vet + acceptance tests + install to ~/.terraform.d/plugins/
```

`make build` installs the binary directly to `~/.terraform.d/plugins/terraform-provider-kong`, which is how it gets picked up by Terraform 0.12-era local plugin resolution.

Cross-platform release binaries are built via GoReleaser (`make release` / `.goreleaser.yml`), targeting linux/darwin/windows on amd64/arm/arm64.

### Testing

Acceptance tests (`TestAcc*`) spin up a real Kong container via the `gokong/containers` package — Docker must be running locally.

```sh
make test      # unit tests only (no Docker required)
make testacc   # acceptance tests only (requires Docker)
make build     # runs goimportscheck + vet + testacc, then installs
```

The default Kong version used by tests is `1.0.2` (set in `kong/provider_test.go`). Override with `KONG_VERSION=<tag>`.

Do not run `TF_ACC=1 make` against a live Kong deployment — the test suite manages its own containerised Kong instance.

### Code conventions

- All resource files must pass `goimports` before commit. Run `make goimports` to auto-fix; `make goimportscheck` to verify.
- Plugin configuration is always passed as a raw JSON string via `config_json`. The `config` map property was removed for Kong ≥ 1.0.0 compatibility — do not reintroduce it.
- `strict_plugins_match` / `strict_match` on `kong_plugin` controls whether Terraform diffs extra keys that Kong injects into plugin config. Default is `false` (tolerant). Be deliberate when changing this on a resource.
- New resources follow the existing CRUD pattern: implement `resourceKong<Name>()` returning a `*schema.Resource`, register it in `provider.go`'s `ResourcesMap`, and add a `_test.go` file with at least one `TestAcc` acceptance test.

### CircleCI / release

The CI pipeline (`.circleci/config.yml`) only builds the binary for `linux/amd64` and stores it as a build artifact — it does not run tests and does not publish releases. Releases are produced by GoReleaser (`.goreleaser.yml`) and require a `GPG_FINGERPRINT` environment variable for checksum signing. The release is created as a **draft** on GitHub.

### Security

- Never commit `.env*` files or any credentials.
- `kong_admin_token` and `kong_api_key` are secrets — do not log, print, or hardcode them.
