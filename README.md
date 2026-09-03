# SignalControl Edge Collector

Dash0's distribution of the OpenTelemetry Collector. It's the OpenTelemetry Collector Contrib build plus Dash0's SignalControl components, which run tail sampling, RED metrics, signal-to-metrics, and telemetry filtering inside your own network before the telemetry is sent to Dash0.

You run this collector yourself, at the edge.

## Pull

```
docker pull ghcr.io/dash0hq/signal-control-edge-collector:<tag>
```

Images are published to the GitHub Container Registry and tagged by version, for example `1.0.0`. There is no `latest` tag, so pick a released version. Both `linux/amd64` and `linux/arm64` are in the manifest.

## Run

The collector needs a config file and two environment variables: `DASH0_AUTH_TOKEN` (your organization's `auth_…` token) and `DASH0_DATASET`. Mount the config at `/etc/otelcol/config.yaml`:

```
docker run --rm --publish 4317:4317 --publish 4318:4318 \
  -v "$(pwd)/config.yaml:/etc/otelcol/config.yaml" \
  -e DASH0_AUTH_TOKEN -e DASH0_DATASET \
  ghcr.io/dash0hq/signal-control-edge-collector:<tag>
```

It receives OTLP on port 4317 (gRPC) and 4318 (HTTP). Point your applications at those ports.

## Configuration

The reference config and a full setup walkthrough live in the SignalControl Edge docs.
