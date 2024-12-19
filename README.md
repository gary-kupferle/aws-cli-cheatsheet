#AWS CLI commands I often use.

### EC2 instances
```bash
# list all instances
aws ec2 describe-instances
```

```bash
# list only instances per a filter
aws ec2 describe-instances --filters "Name=tag:Type, Values=StudentWorkstation"
```

```bash
# list only InstanceId values. ' --output text ' is important to remove the 
# JSON structure chars (braces and brackets)
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Instance:InstanceId}' --output text
```

```bash
# start instances
aws ec2 start-instances --instance-ids "string-id-1" "string-id-2"
```

```bash
# stop instances
aws ec2 stop-instances --instance-ids "string-id-1" "string-id-2"
```

```bash
# get token for curl calls to http://169.254.169.254
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```

```bash
# Get instance metadata (requires setting of $TOKEN, as above) endpoints
curl -v -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
```

```bash
# Get specific piece of metadata, example of internal ipv4 address
curl -v -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/local-ipv4 
```

```bash
# Get instance user data (scripts run at instance startup)
curl -v -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data
```

### Profiles
```bash
# Temporarily switch profiles
export AWS_PROFILE=otherUser
aws s3 ls # Should now list buckets for other_user
```

```bash
# Run a command with a different profile
aws s3 ls --profile yetAnotherUser
```

### General Useful Commands
``` bash
# What region is the default?
aws configure get region
```

```bash# List S3 buckets (and check if your SSO login/environment vars/default creds and profile are set where you think they are
aws s3 ls
```

### API Gateway
```bash
# Get the id's and names of the current account's API Gateway (region and account must be setup previously)
# -E is for 'extended' regex
aws apigateway get-rest-apis | grep -E 'id|name'
```

```bash
# Deploy an API Gateway stage
aws apigateway create-deployment --rest-api-id <api-id> --stage-name <stage-name>
```

```bash
# API Gateway - verify a deployment/list an API's stages
aws apigateway get-stages --rest-api-id <api-id>
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
```

```bash
# Use previous `aws iam list-policies ...` command to set an environment variable and use it in `amplify init ...`
boundary_policy=$(aws iam list-policies --query 'Policies[?PolicyName==`AmplifyPermissionsBoundary`].Arn' --output text)
amplify init --permissions-boundary $boundary_policy
```

```bash
## Amplify commands to init and deploy a front-end React app (used during edx.org online training)
amplify init
```

```bash
amplify status          # will show you what you've added already and if it's locally configured or deployed
```

```bash
amplify add <category>  # will allow you to add features like user login or a backend API
```

```bash
amplify push            # will build all your local backend resources and provision it in the cloud
```

```bash
amplify console         # to open the Amplify Console and view your project status
```

```bash
amplify publish         # will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud 

# Pro tip:
# Try "amplify add api" to create a backend API and then "amplify push" to deploy everything
```

#### Cognito/Amplify UI-API config collaboration
```bash
# If you used `amplify init` in a UI/front-end app, you will have an amplify-meta.json file that you
# can use the following commands on:
APP_CLIENT_ID_WEB=$(jq -r '.auth | .[keys[0]] | .output.AppClientIDWeb' .../ui-project-like-react/amplify/backend/amplify-meta.json)
USER_POOL_ID=$(jq -r '.auth | .[keys[0]] | .output.UserPoolId' .../ui-project-like-react/amplify/backend/amplify-meta.json)

printf "\n\nUserPoolId placeholder value: $USER_POOL_ID\n\nAppClientIDWeb placeholder value: $APP_CLIENT_ID_WEB\n\n"
```

```bash
# if using SAM for lambda development, those env vars can in turn be used to update the template.yaml of the API
as shown below
before=$(cat ~/environment/api-backend-sam/template.yaml | head -41 | tail -7)
update1=$(sed -i "s/REPLACE_WITH_USERPOOL_ID/$USER_POOL_ID/g" .../api-backend-sam/template.yaml)
update2=$(sed -i "s/REPLACE_WITH_APP_CLIENT_ID_WEB/$APP_CLIENT_ID_WEB/g" .../api-backend-sam/template.yaml)
after=$(cat .../api-backend-sam/template.yaml | head -41 | tail -7)

printf "template.yaml --> code BEFORE update:\n\n$before; \n\ntemplate.yaml --> code AFTER update:\n\n $after\n\n"
```
