# ServiceNow Change Request Approval Action

A CloudBees composite action that automates the complete ServiceNow Change Request lifecycle with approval gates.

## Features

- ✅ Creates ServiceNow Change Request
- ✅ Automatically transitions through CR states (Assess → Authorize → Scheduled → Implement → Review)
- ✅ Waits for manual approval in ServiceNow
- ✅ Auto-closes CR on successful approval
- ✅ Returns CR number and approval status for downstream steps

## Usage

```yaml
steps:
  - name: Create and approve ServiceNow CR
    uses: https://github.com/guru-actions/servicenow-change-approval@v1
    with:
      url: ${{ vars.SERVICENOW_URL }}
      username: ${{ secrets.SERVICENOW_USERNAME }}
      password: ${{ secrets.SERVICENOW_PASSWORD }}
      short-description: "Production deployment for release v1.2.3"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `url` | Yes | - | ServiceNow instance URL (e.g., https://dev12345.service-now.com) |
| `username` | Yes | - | ServiceNow username |
| `password` | Yes | - | ServiceNow password |
| `short-description` | Yes | - | Short description for the Change Request |
| `assignment-group` | No | `Change Management` | Assignment group for the Change Request |
| `cr-type` | No | `normal` | Type of CR (normal, emergency, standard) |
| `poll-interval` | No | `1` | Polling interval in minutes |
| `poll-duration` | No | `10` | Maximum polling duration in minutes |
| `close-notes` | No | `Successfully completed` | Notes to add when closing the CR |

## Outputs

| Output | Description |
|--------|-------------|
| `cr-sys-id` | ServiceNow Change Request system ID |
| `cr-number` | ServiceNow Change Request number (e.g., CHG0001234) |
| `approval-status` | Approval status (approved/rejected/timeout) |
| `cr-state` | Final Change Request state |

## Example with Outputs

```yaml
jobs:
  deploy-to-production:
    steps:
      - name: Request ServiceNow approval
        id: servicenow
        uses: https://github.com/guru-actions/servicenow-change-approval@v1
        with:
          url: ${{ vars.SERVICENOW_URL }}
          username: ${{ secrets.SERVICENOW_USERNAME }}
          password: ${{ secrets.SERVICENOW_PASSWORD }}
          short-description: "Deploy squid-ui v${{ needs.build.outputs.VERSION }} to production"
          poll-interval: "2"
          poll-duration: "30"

      - name: Display CR information
        run: |
          echo "Change Request: ${{ steps.servicenow.outputs.cr-number }}"
          echo "Approval Status: ${{ steps.servicenow.outputs.approval-status }}"
          echo "CR State: ${{ steps.servicenow.outputs.cr-state }}"

      - name: Deploy to production
        if: steps.servicenow.outputs.approval-status == 'approved'
        # ... deployment steps
```

## Emergency Changes

```yaml
steps:
  - name: Emergency ServiceNow CR
    uses: https://github.com/guru-actions/servicenow-change-approval@v1
    with:
      url: ${{ vars.SERVICENOW_URL }}
      username: ${{ secrets.SERVICENOW_USERNAME }}
      password: ${{ secrets.SERVICENOW_PASSWORD }}
      short-description: "Emergency hotfix for critical security vulnerability"
      cr-type: "emergency"
      poll-interval: "1"
      poll-duration: "5"
```

## Workflow States

The action automatically transitions the Change Request through these states:

1. **Created** - Initial CR creation
2. **Assess** - Assigned to Change Management team
3. **Authorize** - Authorization review
4. **Scheduled** - Deployment scheduled
5. **Implement** - Implementation in progress
6. **Review** - Awaiting approval (polling starts here)
7. **Closed** - Auto-closed on successful approval

## Configuration Requirements

### Secrets
Configure these secrets in your CloudBees platform:
- `SERVICENOW_USERNAME` - Your ServiceNow username
- `SERVICENOW_PASSWORD` - Your ServiceNow password

### Variables
Configure these variables in your CloudBees platform:
- `SERVICENOW_URL` - Your ServiceNow instance URL

## License

MIT

## Maintainer

Maintained by the guru-actions team.
