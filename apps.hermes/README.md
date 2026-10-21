# Hermes Agent

General-purpose AI agent from Nous Research, deployed as an OpenShell sandbox
instead of a Helm release.

## How it runs

- The Application CR carries `spec.sandbox`; arx-operator-apps creates the
  sandbox through the OpenShell gateway.
- The sandbox runs `hermes serve` on `127.0.0.1:9119` (headless backend). The
  image's s6 entrypoint is bypassed; the sandbox supervisor manages the
  process.
- Filesystem: read-only system paths and `/opt/hermes`, read-write `/tmp` and
  the workdir. Hermes state (`HERMES_HOME`) is `/sandbox/.hermes`, kept across
  stop and start.
- Network: denied by default. The only allowed destination is the
  llm-gateway, bound to the AI credential for the Hermes Python interpreter.
  Anything else the agent tries to reach becomes a proposal that an
  administrator can approve or reject.
- Credential: the agent reads `ARX_LLM_API_KEY` as an OpenShell placeholder;
  the supervisor substitutes the real inference token on gateway requests.

## Model

The install dialog asks for a model for the `llm` slot and recommends
Qwen3.6 35B, deploying it when it is not running yet. On first start the
sandbox writes `/sandbox/.hermes/config.yaml` with a `custom` provider that
points at the llm-gateway and uses the chosen instance as the default model.

Try it from a shell in the sandbox:

```sh
hermes -z "Use your terminal tool to run uname -m and reply with the output."
```

## Resources Required

| Resource | Minimum |
|----------|---------|
| CPU | 1 core |
| Memory | 2 GB |
| Storage | 5 Gi |

The image is about 3.2 GB; pre-pull it on the node if sandbox creation times
out (`k3s crictl pull docker.io/nousresearch/hermes-agent:v0.22.2`).

## Requirements

- An OpenShell gateway configured on arx-operator-apps
  (`controller.openshell.address`).
- Cilium policy enforcement enabled, so the sandbox network fence holds.
- An arm64 node: `inference.binaries` names the arm64 interpreter path.
