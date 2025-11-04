# AWS IAM Identity Center - Inactive User Removal Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![AWS CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-orange.svg)](https://aws.amazon.com/cloudformation/)

Automated solution to remove inactive IAM Identity Center users who haven't signed in for a specified number of days. This CloudFormation template deploys a Lambda function that runs on a schedule to identify and remove inactive users, with optional Slack notifications.

## Features

- ✅ **Automatic Detection**: Identifies users who haven't signed in for a configurable threshold (default: 90 days)
- ✅ **CloudTrail Integration**: Uses CloudTrail events to determine user activity
- ✅ **Safe Removal**: Automatically removes account assignments before deleting users
- ✅ **Slack Notifications**: Optional Slack webhook integration for operation summaries
- ✅ **Scheduled Execution**: Configurable EventBridge schedule (default: monthly)
- ✅ **Comprehensive Logging**: CloudWatch Logs integration with configurable retention

## Architecture

```
┌─────────────────┐
│  EventBridge    │
│  (Scheduler)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Lambda Function│◄──┐
└────────┬────────┘   │
         │           │
         ├───────────►│
         │           │
    ┌────▼────┐  ┌──▼────┐  ┌──────────┐
    │CloudTrail│  │   SSO  │  │Identity  │
    │          │  │  Admin │  │  Store   │
    └──────────┘  └────────┘  └──────────┘
         │
         ▼
    ┌─────────┐
    │  Slack  │ (Optional)
    └─────────┘
```

## Prerequisites

- AWS Account with IAM Identity Center enabled
- AWS Organizations (if using multi-account setup)
- CloudTrail enabled (for tracking user authentication events)
- AWS CLI configured with appropriate permissions
- (Optional) Slack webhook URL for notifications

## Required IAM Permissions

The Lambda execution role requires the following permissions:

- `cloudtrail:LookupEvents` - Check user authentication events
- `organizations:ListAccounts` - List all AWS accounts
- `sso-admin:ListAccountAssignments` - List user account assignments
- `sso-admin:ListPermissionSets` - List permission sets
- `sso-admin:ListInstances` - Get Identity Center instance info
- `sso-admin:DeleteAccountAssignment` - Remove user assignments
- `identitystore:ListUsers` - List Identity Center users
- `identitystore:DeleteUser` - Delete inactive users
- `identitystore:DescribeUser` - Get user details
- `logs:*` - CloudWatch Logs permissions

## Deployment

### Option 1: AWS Console

1. Navigate to CloudFormation console
2. Click "Create Stack" → "With new resources (standard)"
3. Upload the template file: `RemoveInactiveIdentityCenterUsers.yaml`
4. Fill in the parameters:
   - **InactiveDaysThreshold**: Number of days (default: 90)
   - **SlackWebhookURL**: Your Slack webhook URL (optional)
   - **ScheduleExpression**: Cron expression (default: `cron(0 0 1 * ? *)` for monthly)
   - **EnableNotifications**: Enable/disable Slack notifications
5. Review and create the stack

### Option 2: AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --parameters \
    ParameterKey=InactiveDaysThreshold,ParameterValue=90 \
    ParameterKey=SlackWebhookURL,ParameterValue=https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
    ParameterKey=ScheduleExpression,ParameterValue='cron(0 0 1 * ? *)' \
    ParameterKey=EnableNotifications,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### Option 3: AWS SAM

```bash
sam deploy \
  --template-file RemoveInactiveIdentityCenterUsers.yaml \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --parameter-overrides \
    InactiveDaysThreshold=90 \
    SlackWebhookURL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
    ScheduleExpression='cron(0 0 1 * ? *)' \
    EnableNotifications=true \
  --capabilities CAPABILITY_NAMED_IAM
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `InactiveDaysThreshold` | Number | 90 | Number of days of inactivity before a user is considered inactive (1-365) |
| `SlackWebhookURL` | String | `''` | Slack webhook URL for notifications (leave empty to disable) |
| `ScheduleExpression` | String | `cron(0 0 1 * ? *)` | EventBridge schedule expression (cron or rate format) |
| `EnableNotifications` | String | `true` | Enable Slack notifications (requires SlackWebhookURL) |

## Schedule Expressions

### Cron Examples

- **Monthly on 1st day at midnight UTC**: `cron(0 0 1 * ? *)`
- **Weekly on Monday at 2 AM UTC**: `cron(0 2 ? * MON *)`
- **Daily at midnight UTC**: `cron(0 0 * * ? *)`
- **Every 15th of the month at 3 AM UTC**: `cron(0 3 15 * ? *)`

### Rate Examples

- **Every 30 days**: `rate(30 days)`
- **Every 7 days**: `rate(7 days)`

For more information, see [AWS EventBridge Schedule Expressions](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html).

## Slack Integration

### Creating a Slack Webhook

1. Go to [Slack Apps](https://api.slack.com/apps)
2. Create a new app or select existing app
3. Navigate to "Incoming Webhooks"
4. Activate incoming webhooks
5. Add a new webhook to your workspace
6. Copy the webhook URL

### Notification Format

The Lambda function sends notifications with:
- **Color coding**: Green (success), Yellow (partial success), Red (failure)
- **Summary statistics**: Total users checked, inactive users found, deleted users
- **User list**: Names of deleted users
- **Error details**: Any failures during the process

Example notification:
```
IAM Identity Center - Inactive User Removal

Inactive User Removal Summary
• Total users checked: 150
• Inactive users found: 5
• Successfully deleted: 5
• Failed deletions: 0
• Inactivity threshold: 90 days

Deleted Users:
• user1@example.com
• user2@example.com
• user3@example.com
```

## How It Works

1. **Scheduled Trigger**: EventBridge rule triggers the Lambda function on the specified schedule
2. **Instance Discovery**: Lambda retrieves the IAM Identity Center instance ARN and Identity Store ID
3. **User Listing**: All users in Identity Center are retrieved
4. **Activity Check**: For each user, CloudTrail is queried for `UserAuthentication` events within the threshold period
5. **Assignment Removal**: Inactive users' account assignments are removed first
6. **User Deletion**: Users are deleted from Identity Center
7. **Notification**: Summary is sent to Slack (if configured)

## Important Notes

⚠️ **External Identity Providers**: If you use an external identity provider (e.g., Active Directory, Okta), you must also delete users from the external provider. This solution only removes users from AWS IAM Identity Center.

⚠️ **User Reprovisioning**: After deletion, users cannot be automatically reprovisioned. They must be re-added manually.

⚠️ **Testing**: Before deploying to production, test with a short schedule (e.g., `rate(5 minutes)`) and verify the behavior.

⚠️ **CloudTrail**: Ensure CloudTrail is enabled and logging `UserAuthentication` events. Without this, all users will appear inactive.

## Monitoring

### CloudWatch Logs

View logs in CloudWatch Logs under:
```
/aws/lambda/RemoveInactiveIdentityCenterUsers-<region>
```

### CloudWatch Metrics

The Lambda function automatically reports:
- Invocations
- Duration
- Errors
- Throttles

### Troubleshooting

Common issues and solutions:

| Issue | Solution |
|-------|----------|
| All users being deleted | Verify CloudTrail is logging `UserAuthentication` events |
| Permission errors | Check IAM role has all required permissions |
| Timeout errors | Increase Lambda timeout (default: 900 seconds) |
| No Slack notifications | Verify webhook URL is correct and notifications are enabled |

## Cost Estimation

- **Lambda**: ~$0.20 per million requests (free tier: 1M requests/month)
- **CloudWatch Logs**: ~$0.50 per GB ingested (free tier: 5 GB/month)
- **EventBridge**: Free for scheduled rules
- **CloudTrail**: Free for management events (if enabled)

**Estimated monthly cost**: < $1 for typical usage

## Security Considerations

- The Lambda function uses least-privilege IAM permissions
- Slack webhook URLs are stored as encrypted environment variables
- CloudWatch Logs are retained for 90 days (configurable)
- All operations are logged for audit purposes

## Updating the Stack

To update parameters:

```bash
aws cloudformation update-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --use-previous-template \
  --parameters \
    ParameterKey=InactiveDaysThreshold,ParameterValue=60 \
    ParameterKey=SlackWebhookURL,UsePreviousValue=true \
    ParameterKey=ScheduleExpression,UsePreviousValue=true \
    ParameterKey=EnableNotifications,UsePreviousValue=true
```

## Uninstalling

To remove all resources:

```bash
aws cloudformation delete-stack \
  --stack-name RemoveInactiveIdentityCenterUsers
```

**Note**: This will delete the Lambda function, EventBridge rule, IAM role, and CloudWatch Log Group.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Author

**Vinayak Singh**

## Acknowledgments

- Based on AWS re:Post article: [How do I automatically remove inactive IAM Identity Center users?](https://repost.aws/knowledge-center/auto-remove-inactive-iam-identity-center-users)
- AWS IAM Identity Center documentation
- AWS Lambda and EventBridge services

## Support

For issues, questions, or contributions, please open an issue on GitHub.

## Changelog

### Version 1.0.0
- Initial release
- Automatic inactive user detection and removal
- Slack notification integration
- Configurable schedule and thresholds
- CloudWatch Logs integration
