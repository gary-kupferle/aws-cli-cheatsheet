#AWS CLI commands I often use.

### EC2 instances
```bash
# list all instances
aws ec2 describe-instances

# list only instances per a filter
aws ec2 describe-instances --filters "Name=tag:Type, Values=StudentWorkstation"

# list only InstanceId values. ' --output text ' is important to remove the 
# JSON structure chars (braces and brackets)
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Instance:InstanceId}' --output text

# start and stop instances
aws ec2 start-instances --instance-ids "string-id-1" "string-id-2"
aws ec2 stop-instances --instance-ids "string-id-1" "string-id-2"
```

### Profiles
```bash
# Temporarily switch profiles
export AWS_PROFILE=otherUser
aws s3 ls # Should now list buckets for other_user

# Run a command with a different profile
aws s3 ls --profile yetAnotherUser
```
