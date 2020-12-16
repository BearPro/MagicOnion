## Table of Contents

* [server ops](#server-ops)
* [client ops](#client-ops)

## Basics
### build

```shell
# linux
dotnet publish Benchmark.Server/Benchmark.Server.csproj -c Release -r linux-x64 -p:PublishSingleFile=true --no-self-contained -o out/linux/server
dotnet publish Benchmark.Client/Benchmark.Client.csproj -c Release -r linux-x64 -p:PublishSingleFile=true --no-self-contained -o out/linux/client

# win
dotnet publish Benchmark.Server/Benchmark.Server.csproj -c Release -r win-x64 -p:PublishSingleFile=true --no-self-contained -o out/win/server
dotnet publish Benchmark.Client/Benchmark.Client.csproj -c Release -r win-x64 -p:PublishSingleFile=true --no-self-contained -o out/win/client
```

### run

run server
```shell
ASPNETCORE_ENVIRONMENT=Development ./out/linux/server/Benchmark.Server
ASPNETCORE_ENVIRONMENT=Production sudo ./out/linux/server/Benchmark.Server
```

run client

```shell
./out/linux/server/Benchmark.Client
```

### push binary to s3 bucket

```
BUCKET=bench-magiconion-s3-bucket-5c7e45b
aws s3 sync ./out/linux/server/ s3://${BUCKET}/assembly/linux/server/
aws s3 sync ./out/linux/client/ s3://${BUCKET}/assembly/linux/client/
```

## server ops

update binary

```
dotnet publish Benchmark.Server/Benchmark.Server.csproj -c Release -r linux-x64 -p:PublishSingleFile=true --no-self-contained -o out/linux/server
BUCKET=bench-magiconion-s3-bucket-5c7e45b
aws s3 sync ./out/linux/server/ s3://${BUCKET}/assembly/linux/server/
instanceId=$(aws ssm describe-instance-information --output json --filters Key=tag-key,Values=bench --filters=Key=PingStatus,Values=Online | jq -r ".InstanceInformationList[].InstanceId")
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://download_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations --command-id "${commandId}" --details | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

get instanceid

```shell
instanceId=$(aws ssm describe-instance-information --output json --filters Key=tag-key,Values=bench --filters=Key=PingStatus,Values=Online | jq -r ".InstanceInformationList[].InstanceId")
```

download

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://download_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

register to systemd

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://register_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

run

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://run_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

logs

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://log_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

stop

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://stop_server.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

## client ops

update binary

```
dotnet publish Benchmark.Client/Benchmark.Client.csproj -c Release -r linux-x64 -p:PublishSingleFile=true --no-self-contained -o out/linux/client
BUCKET=bench-magiconion-s3-bucket-5c7e45b
aws s3 sync ./out/linux/client/ s3://${BUCKET}/assembly/linux/client/
instanceId=$(aws ssm describe-instance-information --output json --filters Key=tag-key,Values=bench --filters=Key=PingStatus,Values=Online | jq -r ".InstanceInformationList[].InstanceId")
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://download_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations --command-id "${commandId}" --details | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://register_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations --command-id "${commandId}" --details | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://run_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations --command-id "${commandId}" --details | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

get instanceid

```shell
# todo add client TagKey
instanceId=$(aws ssm describe-instance-information \
    --output json \
    --filters Key=tag-key,Values=bench \
    --filters=Key=PingStatus,Values=Online \
    | jq -r ".InstanceInformationList[].InstanceId")
```

download

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://download_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

register to systemd

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://register_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

run

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://run_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

run (CLI)

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://run_client_cli.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```


logs

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://log_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```

stop

```shell
commandId=$(aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=InstanceIds,Values=${instanceId}" --cli-input-json file://stop_client.json --output json | jq -r ".Command.CommandId")
aws ssm list-command-invocations \
	--command-id "${commandId}" \
	--details \
    | jq -r ".CommandInvocations[].Status, .CommandInvocations[].CommandPlugins[].Output"
```