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

``` yaml
with:
    template: push
```

#### `status` (required)

The `status` must be defined. It can either be the current job status using:

``` yaml
with:
    status: ${{ job.status }}
```

or a hardcoded custom status such as "starting" or "in progress":

``` yaml
with:
    status: in progress
```

#### `slack_webhook_url` (optional)

Only required if the `SLACK_WEBHOOK_URL` environment variable is not set.

``` yaml
with:
    slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### `steps` (optional)

The individual status of job steps can be included in the Slack message using:

``` yaml
with:
    steps: ${{ steps }}
```

Note: Only steps that have a "step id" will be reported on. See example below.

#### `channel` (optional)

To override the channel or to send the Slack message to an individual use:

``` yaml
with:
    channel: '#workflows'
```

#### `logo_url` (optional)

Logo displayed in slack message.

``` yaml
with:
    logo_url: 'https://s3-us-west-2.amazonaws.com/slack-files2/bot_icons/2023-01-09/4618522117268_48.png'
```

Default logo is: ![Default logo](https://s3-us-west-2.amazonaws.com/slack-files2/bot_icons/2023-01-09/4618522117268_48.png "Default logo")

### Template requirements

#### `pr` template

The `pr` template require 2 environment variables (`env`) `PR_URL` AND `PR_NUMBER`.

The action [pull-request@v2](https://github.com/repo-sync/pull-request) create an new PR and output the url and number.

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
        uses: steplix/cicd-notify@1.0.0
        with:
          template: pr
          channel: '#back-test-pipes'
          status: ${{ job.status }}
          steps: ${{ toJson(steps) }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          PR_URL: ${{ steps.pr.outputs.pr_url }}
          PR_NUMBER: ${{ steps.pr.outputs.pr_number }}
```

#### `release` template

The `release` template require `NEW_TAG` environment variables (`env`).

``` yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: slack - GitHub Actions Slack integration
        uses: steplix/cicd-notify@1.0.0
        with:
          template: release
          channel: '#back-test-pipes'
          status: ${{ job.status }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          NEW_TAG: '1.0.1'
```
