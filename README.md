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

### General Useful Commands
``` bash
# What region is the default?
aws configure get region
```

### Cloud Watch
``` bash
# Tail follow logs from a lambda function
# Last argument is Log Group name from top of page in AWS console
aws logs tail --follow /aws/lambda/kups-grid-maker-for-edx-learner
```

### Amplify and Cognito
```bash
# Get the Amplify boundary policyies (if any exist)
aws iam list-policies --query 'Policies[?PolicyName==`AmplifyPermissionsBoundary`].Arn' --output text

# Use previous `aws iam list-policies ...` command to set an environment variable and use it in `amplify init ...`
boundary_policy=$(aws iam list-policies --query 'Policies[?PolicyName==`AmplifyPermissionsBoundary`].Arn' --output text)
amplify init --permissions-boundary $boundary_policy

## Amplify commands to init and deploy a front-end React app (used during edx.org online training)
amplify init
amplify status          # will show you what you've added already and if it's locally configured or deployed
amplify add <category>  # will allow you to add features like user login or a backend API
amplify push            # will build all your local backend resources and provision it in the cloud
amplify console         # to open the Amplify Console and view your project status
amplify publish         # will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud 

# Pro tip:
# Try "amplify add api" to create a backend API and then "amplify push" to deploy everything

# If you used `amplify init` in a UI/front-end app, you will have an amplify-meta.json file that you
# can use the following commands on:
APP_CLIENT_ID_WEB=$(jq -r '.auth | .[keys[0]] | .output.AppClientIDWeb' .../ui-project-like-react/amplify/backend/amplify-meta.json)

USER_POOL_ID=$(jq -r '.auth | .[keys[0]] | .output.UserPoolId' .../ui-project-like-react/amplify/backend/amplify-meta.json)

printf "\n\nUserPoolId placeholder value: $USER_POOL_ID\n\nAppClientIDWeb placeholder value: $APP_CLIENT_ID_WEB\n\n"
# if using SAM for lambda development, those env vars can in turn be used to update the template.yaml of the API
as shown below
before=$(cat ~/environment/api-backend-sam/template.yaml | head -41 | tail -7)
update1=$(sed -i "s/REPLACE_WITH_USERPOOL_ID/$USER_POOL_ID/g" .../api-backend-sam/template.yaml)
update2=$(sed -i "s/REPLACE_WITH_APP_CLIENT_ID_WEB/$APP_CLIENT_ID_WEB/g" .../api-backend-sam/template.yaml)
after=$(cat .../api-backend-sam/template.yaml | head -41 | tail -7)

printf "template.yaml --> code BEFORE update:\n\n$before; \n\ntemplate.yaml --> code AFTER update:\n\n $after\n\n"
# end of Cognito/Amplify UI-API config collaboration


```
