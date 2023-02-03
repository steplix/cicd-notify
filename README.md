# cicd-notify

Github action to send notifications via slack with different templates. \
This action call the action [act10ns/slack@v2.0.0](https://github.com/act10ns/slack).

## Configuration

### Environment Variables (`input`)

#### `SLACK_WEBHOOK_URL` (required)

Create a Slack Webhook URL using either the
[Incoming Webhooks App](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks?next_id=0)
(preferred) or by attaching an incoming webhook to an existing
[Slack App](https://api.slack.com/apps) (beware, channel override not possible
when using a Slack App):

``` yaml
env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Input Parameters (`with`)

#### `template` (required)

Pass the name of the to use, available templates:

1. pr
2. push
3. release
4. start
5. tag

``` yaml
with:
    template: push
```

#### `slack_webhook_url` (optional)

Only required if the `SLACK_WEBHOOK_URL` environment variable is not set.

``` yaml
with:
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### `status` (optional)

By default value, is the current job status `${{job.status}}`. If required, can be sent a custom hardcoded value such as "starting" or "in progress":

``` yaml
with:
    status: in progress
```

#### `channel` (optional)

To override the channel or to send the Slack message to an individual use:

``` yaml
with:
    channel: '#workflows'
```

## IMPORTANT

To use the `pr` template, keep in mind that it must be within the same job after a step with id `pr` and that it has the following outputs:

``` json
{
    "outputs": {
        "pr_url": "https://github.com/steplix/jira-integration/pull/12",
        "pr_number": "12"
    }
}
```

### `Example`

The action [pull-request@v2](https://github.com/repo-sync/pull-request) create an new PR and send the required output.

``` yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: create-pr
        id: pr
        uses: repo-sync/pull-request@v2
        with:
          destination_branch: testing
          github_token: ${{ secrets.GITHUB_TOKEN }}

      - name: slack - GitHub Actions Slack integration
        uses: steplix/cicd-notify@0.0.18
        with:
          template: pr
          channel: '#back-test-pipes'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```
