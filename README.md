#AWS CLI commands I often use.

### EC2 instances
```bash
# list all instances
aws ec2 describe-instances

# list only instances per a filter
aws ec2 describe-instances --filters "Name=tag:Type, Values=StudentWorkstation"

# list only InstanceId values. ' --output text ' is important to remove the JSON structure chars (braces and brackets)
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Instance:InstanceId}' --output text
u

# start and stop instances
aws ec2 start-instances --instance-ids "string-id-1" "string-id-2"
aws ec2 stop-instances --instance-ids "string-id-1" "string-id-2"
```
