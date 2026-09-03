# SignalControl Edge

Dash0's SignalControl Edge distribution. It runs tail sampling, RED metrics, signal-to-metrics, and telemetry filtering inside your own network before the telemetry is sent to Dash0.

It ships two images, both published to the GitHub Container Registry:

- **[SignalControl Edge Collector](#signalcontrol-edge-collector)** (`ghcr.io/dash0hq/signal-control-edge-collector`) — the collector you run at the edge.
- **[Edge Proxy](#edge-proxy)** (`ghcr.io/dash0hq/edge-proxy`) — an optional proxy that holds one upstream connection per site and fans the results out to your collectors.

## SignalControl Edge Collector

Dash0's distribution of the OpenTelemetry Collector. It's the OpenTelemetry Collector Contrib build plus Dash0's SignalControl components, which run tail sampling, RED metrics, signal-to-metrics, and telemetry filtering inside your own network before the telemetry is sent to Dash0.

You run this collector yourself, at the edge.

### Pull

```
docker pull ghcr.io/dash0hq/signal-control-edge-collector:<tag>
```

Images are published to the GitHub Container Registry and tagged by version, for example `1.0.0`. There is no `latest` tag, so pick a released version. Both `linux/amd64` and `linux/arm64` are in the manifest.

### Run

The collector needs a config file and two environment variables: `DASH0_AUTH_TOKEN` (your organization's `auth_…` token) and `DASH0_DATASET`. Mount the config at `/etc/otelcol/config.yaml`:

```
docker run --rm --publish 4317:4317 --publish 4318:4318 \
  -v "$(pwd)/config.yaml:/etc/otelcol/config.yaml" \
  -e DASH0_AUTH_TOKEN -e DASH0_DATASET \
  ghcr.io/dash0hq/signal-control-edge-collector:<tag>
```

It receives OTLP on port 4317 (gRPC) and 4318 (HTTP). Point your applications at those ports.

### Configuration

The reference config and a full setup walkthrough live in the SignalControl Edge docs.

## Edge Proxy

A stateless proxy you run in your own network. It is optional but recommended at scale: instead of each collector connecting to Dash0 directly, the proxy holds one upstream connection per site and fans the results out to the collectors. It runs two independent pipelines, each enabled separately:

- **Tail-sampling decisions** (`upstream.tailSampling.enabled`, default `true`). Forwards sampling reports from collectors to Dash0 and fans decisions back, so decisions cross the internet once.
- **Edge-settings fan-out** (`upstream.edgeSettings.enabled`, default `false`). Polls Dash0 once per proxy instance and broadcasts the settings snapshot to subscribed collectors over gRPC, so N collectors receive one broadcast per change instead of N polls.

Enabling neither pipeline is rejected at startup.

### Pull

```
docker pull ghcr.io/dash0hq/edge-proxy:<tag>
```

Images are published to the GitHub Container Registry and tagged by version, for example `1.0.0`. There is no `latest` tag, so pick a released version. Both `linux/amd64` and `linux/arm64` are in the manifest.

### Run

The proxy needs a config file. Mount it at `/etc/dash0/edge-proxy.yaml`:

```
docker run --rm --name edge-proxy --publish 8011:8011 --publish 8012:8012 \
  -v "$(pwd)/edge-proxy.yaml:/etc/dash0/edge-proxy.yaml" \
  ghcr.io/dash0hq/edge-proxy:<tag>
```

It listens for collectors on port 8011 (gRPC) and serves internal admin and pprof on 8012. Point your collectors at port 8011.

### Configuration

The two pipelines are configured independently. Every setting also maps to an environment variable, for example `upstream.edgeSettings.address` maps to `UPSTREAM_EDGESETTINGS_ADDRESS`. The full config reference and the environment-variable table live in the SignalControl Edge docs.
