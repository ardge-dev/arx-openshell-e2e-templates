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
- A model served by the llm-gateway under the name in `ARX_LLM_MODEL`.
