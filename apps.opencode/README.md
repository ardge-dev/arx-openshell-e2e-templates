# OpenCode

AI coding agent deployed as an OpenShell sandbox instead of a Helm release.

## How it runs

- The Application CR carries `spec.sandbox`; arx-operator-apps creates the
  sandbox through the OpenShell gateway.
- Filesystem: read-only system paths, read-write `/tmp` and the workdir.
- Network: denied by default. The only allowed destination is the
  llm-gateway, bound to the AI credential for `/usr/local/bin/opencode`.
- Credential: the agent reads `ARX_LLM_API_KEY` as an OpenShell placeholder;
  the supervisor substitutes the real inference token on gateway requests.

## Model

The install dialog asks for a model for the `llm` slot and recommends
Qwen3.6 35B, deploying it when it is not running yet. The chosen instance
reaches the sandbox as `ARX_LLM_MODEL` and is OpenCode's default model.

To use other models deployed on ARX, list them under the `arx` provider in
an OpenCode project config (`opencode.json`); OpenCode does not list the
gateway's models by itself. Files under the workdir `/sandbox` are kept
across stop and start. Listed models appear in `/models` and can be picked
per run with `opencode run -m arx/<instance>`.

## Resources Required

| Resource | Minimum |
|----------|---------|
| CPU | 1 core |
| Memory | 1 GB |
| Storage | 5 Gi |

## Requirements

- An OpenShell gateway configured on arx-operator-apps
  (`controller.openshell.address`).
- Cilium policy enforcement enabled, so the sandbox network fence holds.
