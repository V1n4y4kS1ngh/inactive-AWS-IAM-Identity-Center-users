# Examples

This document provides practical examples for deploying and using the AWS IAM Identity Center Inactive User Removal solution.

## Basic Deployment

### Minimal Setup (Console Only)

Deploy with default settings (90 days threshold, monthly schedule, no Slack):

```bash
aws cloudformation create-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

## Advanced Configurations

### Weekly Monitoring

Monitor and remove inactive users weekly:

```bash
aws cloudformation create-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --parameters \
    ParameterKey=ScheduleExpression,ParameterValue='cron(0 2 ? * MON *)' \
    ParameterKey=InactiveDaysThreshold,ParameterValue=90 \
  --capabilities CAPABILITY_NAMED_IAM
```

### 30-Day Threshold with Slack

Remove users inactive for 30 days with Slack notifications:

```bash
aws cloudformation create-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --parameters \
    ParameterKey=InactiveDaysThreshold,ParameterValue=30 \
    ParameterKey=SlackWebhookURL,ParameterValue=https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
    ParameterKey=EnableNotifications,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### Daily Execution for Testing

**⚠️ Use only for testing!**

```bash
aws cloudformation create-stack \
  --stack-name RemoveInactiveIdentityCenterUsers-Test \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --parameters \
    ParameterKey=ScheduleExpression,ParameterValue='cron(0 10 * * ? *)' \
    ParameterKey=InactiveDaysThreshold,ParameterValue=90 \
  --capabilities CAPABILITY_NAMED_IAM
```

## Manual Lambda Invocation

### Test Execution

Invoke the Lambda function manually for testing:

```bash
aws lambda invoke \
  --function-name RemoveInactiveIdentityCenterUsers-<region> \
  --payload '{}' \
  response.json

cat response.json
```

### Scheduled Execution Test

Use EventBridge to trigger immediately:

```bash
aws events put-rule \
  --name test-inactive-users \
  --schedule-expression "rate(5 minutes)"

aws events put-targets \
  --rule test-inactive-users \
  --targets "Id"="1","Arn"="arn:aws:lambda:REGION:ACCOUNT:function:RemoveInactiveIdentityCenterUsers-REGION"
```

## Monitoring Examples

### View Recent Logs

```bash
aws logs tail /aws/lambda/RemoveInactiveIdentityCenterUsers-<region> --follow
```

### Check Last Execution

```bash
aws lambda get-function \
  --function-name RemoveInactiveIdentityCenterUsers-<region> \
  --query 'Configuration.LastModified'
```

### View CloudWatch Metrics

```bash
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=RemoveInactiveIdentityCenterUsers-<region> \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 3600 \
  --statistics Sum
```

## Update Examples

### Change Threshold

Update inactivity threshold to 60 days:

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

### Add Slack Notifications

Add Slack notifications to existing deployment:

```bash
aws cloudformation update-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --use-previous-template \
  --parameters \
    ParameterKey=InactiveDaysThreshold,UsePreviousValue=true \
    ParameterKey=SlackWebhookURL,ParameterValue=https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
    ParameterKey=ScheduleExpression,UsePreviousValue=true \
    ParameterKey=EnableNotifications,ParameterValue=true
```

### Change Schedule

Update to run weekly instead of monthly:

```bash
aws cloudformation update-stack \
  --stack-name RemoveInactiveIdentityCenterUsers \
  --use-previous-template \
  --parameters \
    ParameterKey=InactiveDaysThreshold,UsePreviousValue=true \
    ParameterKey=SlackWebhookURL,UsePreviousValue=true \
    ParameterKey=ScheduleExpression,ParameterValue='cron(0 2 ? * MON *)' \
    ParameterKey=EnableNotifications,UsePreviousValue=true
```

## Troubleshooting Examples

### Check Lambda Function Status

```bash
aws lambda get-function-configuration \
  --function-name RemoveInactiveIdentityCenterUsers-<region>
```

### View Recent Errors

```bash
aws logs filter-log-events \
  --log-group-name /aws/lambda/RemoveInactiveIdentityCenterUsers-<region> \
  --filter-pattern "ERROR" \
  --start-time $(date -u -d '1 day ago' +%s)000
```

### Verify IAM Permissions

```bash
aws iam get-role-policy \
  --role-name RemoveInactiveIdentityCenterUsers-Role-<region> \
  --policy-name RemoveInactiveUsersPolicy
```

### Test CloudTrail Access

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=UserAuthentication \
  --max-results 1
```

## Multi-Account Setup

### StackSet Deployment

Deploy to multiple accounts using CloudFormation StackSets:

```bash
aws cloudformation create-stack-set \
  --stack-set-name RemoveInactiveIdentityCenterUsers \
  --template-body file://RemoveInactiveIdentityCenterUsers.yaml \
  --parameters \
    ParameterKey=InactiveDaysThreshold,ParameterValue=90 \
    ParameterKey=SlackWebhookURL,ParameterValue=https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
  --capabilities CAPABILITY_NAMED_IAM

aws cloudformation create-stack-instances \
  --stack-set-name RemoveInactiveIdentityCenterUsers \
  --accounts '["123456789012","987654321098"]' \
  --regions '["us-east-1","us-west-2"]'
```

## Cost Optimization

### Reduce Lambda Timeout

If your user base is small, reduce timeout to save costs:

Edit the template and change:
```yaml
Timeout: 600  # Reduce from 900 to 600 seconds
```

### Adjust Log Retention

Reduce CloudWatch Logs retention:

```bash
aws logs put-retention-policy \
  --log-group-name /aws/lambda/RemoveInactiveIdentityCenterUsers-<region> \
  --retention-in-days 30
```

## Integration Examples

### Combine with SNS

Send notifications to SNS topic instead of Slack:

1. Create SNS topic
2. Modify Lambda function to publish to SNS
3. Subscribe email/SMS to SNS topic

### Combine with S3 Export

Export deleted user list to S3:

Add S3 permissions to Lambda role and modify function to write report to S3.

## Security Best Practices

### Use Secrets Manager for Slack Webhook

Store Slack webhook in AWS Secrets Manager:

```bash
aws secretsmanager create-secret \
  --name identity-center/slack-webhook \
  --secret-string "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# Update Lambda to retrieve from Secrets Manager
```

### Enable VPC Endpoints

If Lambda is in VPC, ensure VPC endpoints for:
- CloudTrail
- SSO Admin
- Identity Store
- Organizations

