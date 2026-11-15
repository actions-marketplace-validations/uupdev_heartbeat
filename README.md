# You Up? Heartbeat Action

You Up? is an uptime monitoring service that checks your websites, APIs, servers and cron jobs from several locations and alerts you by email, Slack, WhatsApp, SMS and more, only when an outage is confirmed.

This GitHub Action sends a Heartbeat signal to [You Up?](https://uup.dev/) when a scheduled workflow or deployment job succeeds. It uses `curl` with zero runtime dependencies.

## How it works

A Heartbeat Monitor watches a job that runs on its own schedule. Place this Action as the last step of your job. GitHub Actions runs each step only when every step before it succeeded. If an earlier step fails, this step does not run, the Heartbeat Monitor receives no signal, and You Up? alerts you once the Grace period runs out.

If the Heartbeat URL is empty or unset (for example, on a pull request from a fork or in a local dry run), the Action skips the call and exits cleanly. The Action masks the URL in GitHub Actions and never prints it to the workflow log.

## Usage

1. Create a Heartbeat Monitor in [You Up?](https://uup.dev/) and copy its Heartbeat URL.
2. Open your GitHub repository settings and save the URL as an Actions secret named `UUP_HEARTBEAT_URL`.
3. Add `uupdev/heartbeat@v1` as the final step of your job:

```yaml
name: Nightly backup

on:
  schedule:
    - cron: "0 3 * * *"

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./backup.sh
      - uses: uupdev/heartbeat@v1
        with:
          url: ${{ secrets.UUP_HEARTBEAT_URL }}
```

GitHub Actions reads workflow `cron` expressions in UTC. Unless your Project timezone in You Up? is UTC, enter the schedule in your Project timezone when you configure the Heartbeat Monitor.

### Reading from an environment variable

You can also pass `UUP_HEARTBEAT_URL` in the job or step environment without `with`:

```yaml
      - uses: uupdev/heartbeat@v1
        env:
          UUP_HEARTBEAT_URL: ${{ secrets.UUP_HEARTBEAT_URL_DEPLOY }}
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `url` | `""` | Heartbeat URL from You Up?. Falls back to the `UUP_HEARTBEAT_URL` environment variable. If both are empty, the step skips cleanly. |
| `timeout` | `"10"` | Maximum time in seconds for the HTTP request. |
| `retries` | `"3"` | Number of retries on transient network or `5xx` errors. |
| `fail-on-error` | `"false"` | Set to `"true"` to fail the step when the Heartbeat request does not return `2xx`. By default, a failed request logs a warning and never fails your job. |

## Outputs

| Output | Description |
| --- | --- |
| `status` | `sent`, `skipped`, or `failed`. |

## Links

- [Website](https://uup.dev/)
- [Heartbeat Monitors documentation](https://docs.uup.dev/monitors/heartbeat/)
- [Heartbeat examples](https://docs.uup.dev/monitors/heartbeat-examples/)
- [AI assistants (MCP)](https://docs.uup.dev/api/mcp/)

## License

[MIT](LICENSE)
