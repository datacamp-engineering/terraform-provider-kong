# terraform-provider-kong

Fork of the open-source Kong Terraform provider (`kevholditch/terraform-provider-kong`) patched for DataCamp's Kong version and plugin set.

See [README.md](./README.md) for full documentation.

## Technology stack

- N/A — this is a provider binary, not a module

## Security guardrails

- Never commit .env* files, API keys, access keys, or tokens
- Never output sensitive values without `sensitive = true`
- Sensitive outputs:
No sensitive outputs identified — but mark any future outputs containing credentials with `sensitive = true`.
- Follow least-privilege for IAM policies — no wildcard actions unless justified with a comment

## Validation

```bash
# Format check
terraform fmt -check -recursive

# Validate configuration
terraform validate

# Plan (requires credentials)
terraform plan
```

## Terraform conventions

- All `variable` blocks must have a `description`
- All `output` blocks must have a `description`
- Use `snake_case` for resource and variable names; `kebab-case` for AWS tags
- Pin provider versions explicitly in `versions.tf` — no open-ended `>= x` constraints
- Add `lifecycle` blocks with inline comments explaining why they exist
- Update `required_version` to `>= 1.5` — the concourse-runner supports Terraform 1.x (upgrade from 1.1.4 in progress)

## What not to do

- Don't apply upstream PRs without verifying compatibility with DataCamp's Kong version
- Don't commit Kong admin API credentials
