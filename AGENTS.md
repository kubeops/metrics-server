# AGENTS.md

This file provides guidance to coding agents (e.g. Claude Code, claude.ai/code) when working with code in this repository.

## Repository purpose

The AppsCode/kubeops fork of [sigs.k8s.io/metrics-server](https://github.com/kubernetes-sigs/metrics-server) — the Kubernetes Metrics Server. Collects container resource metrics from Kubelets and exposes them through the apiserver Metrics API so HPA / VPA / `kubectl top` can use them. **For autoscaling only**: not a general-purpose monitoring backend.

The Go module path stays at upstream: `sigs.k8s.io/metrics-server`. The git remote is `kubeops/storage-metrics-server` (despite the folder name `metrics-server`) — that's a historical naming artifact. Module imports must use the upstream path.

The produced binary is `metrics-server`.

## Architecture (upstream layout)

- `cmd/metrics-server/`:
  - `metrics-server.go` — entry point.
  - `app/` — server bootstrap, flag parsing, manager assembly.
- `pkg/`:
  - `server/` — the aggregated apiserver wiring.
  - `scraper/` — Kubelet metrics scraping.
  - `storage/` — in-memory metrics store.
  - `api/` — Metrics API surface (`v1beta1` `PodMetrics`, `NodeMetrics`).
  - `utils/` — shared helpers.
- `charts/metrics-server/` — Helm chart for installing the server.
- `manifests/` — kustomize-style install manifests.
- `Dockerfile` — release image.
- `test/` — integration and e2e tests.
- `docs/`, `FAQ.md`, `KNOWN_ISSUES.md`, `SECURITY.md`, `code-of-conduct.md` — upstream docs.
- `Makefile` + `scripts/` — upstream Makefile harness (local Go toolchain, no AppsCode Docker wrapper).
- `cloudbuild.yaml`, `skaffold.yaml` — CI / dev-loop integrations.

## Common commands

This repo uses the upstream `sigs.k8s.io/metrics-server` Makefile (local Go toolchain). Consult `Makefile` and `scripts/` for the exact target set. Selected commonly-used targets:

- `make build` — Go build.
- `make test` — Go tests.
- `make verify` — lint / verify.

Run a single Go test:

```
go test ./pkg/scraper/... -run TestName -v
```

## Conventions

- Module path is `sigs.k8s.io/metrics-server` (**upstream**); imports must use that, not the kubeops/storage-metrics-server URL.
- **Upstream-tracking** fork. Prefer rebasing onto upstream over diverging. The fork exists for AppsCode-specific install bundles; isolate any code patches so they replay cleanly.
- License: Apache-2.0 (`LICENSE`, `OWNERS`, `SECURITY.md`).
- Sign off commits (`git commit -s`); contributions follow `CONTRIBUTING.md`.
- The Metrics API surface (`pkg/api/`) is **stable** kubernetes API. Do not break the wire format. New fields go through SIG Instrumentation upstream first.
- Per the upstream README: this server is **not** a monitoring source. Scope new features tightly to autoscaling use cases; don't grow it into a metrics pipeline.
- Charts / manifests live in `charts/metrics-server/` and `manifests/` — keep them in sync with the binary's flag surface.
