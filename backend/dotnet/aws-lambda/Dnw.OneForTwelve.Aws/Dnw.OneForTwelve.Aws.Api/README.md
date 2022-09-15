# Intro

Amazon aws lambda implementation of the game api.

# Deployment

After initial setup (see below) you can create / update the aws lambda with this cli command:

```
dotnet lambda deploy-serverless --stack-name Dnw-OneForTwelve-Aws-Api --s3-bucket dnw-templates-2022
```

And to test your changes:

```
curl https://8tvctbmdz9.execute-api.ap-southeast-1.amazonaws.com/
```

## Initial setup

Install aws cli. Use brew on mac with apple silicon. When following the instructions here: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html you will send end up with the x64 version:

```
brew install awscli
```

In the aws console create a new user "Administrator" and select programmatic access. Save the api & access key. Then in a terminal window run:

```
aws configure
```

And follow the instructions.

Create s3 bucket using the aws cli:

```
aws s3api create-bucket --bucket dnw-templates-2022 --create-bucket-configuration LocationConstraint=ap-southeast-1
```

# Initial creation

```
dotnet new webapi -minimal -o Dnw.OneForTwelve.Aws.Api
```

Note that the assemblyname in the project file has to be set to bootstrap.

# Optional configuration

At the moment running on arm64 aws graviton is about 20% cheaper.

To use arm64 instead of x86_64:

In aws-lambda-tools-defaults.json change these setings:

```
"msbuild-parameters": "--self-contained true --runtime linux-arm64",
```

And in the serverless.template file change these settings:

```
"Architectures": [ "arm64" ]
```

Also in the projectfile add the following conditional ItemGroup:

```
<ItemGroup Condition="'$(RuntimeIdentifier)' == 'linux-arm64'">
  <RuntimeHostConfigurationOption Include="System.Globalization.AppLocalIcu" Value="68.2.0.9" />
  <PackageReference Include="Microsoft.ICU.ICU4C.Runtime" Version="68.2.0.9" />
</ItemGroup>
```

Without this you will see errors like this in the CloudWatch logs: 'Cannot get symbol ucol_setMaxVariable_50 from libicui18n'.

More info on this issue can be found here: https://github.com/normj/LambdaNETCoreSamples/tree/master/ArmLambdaFunction

# Issues

The alternative way to deploy a lambda function without aws cloud formation is to use 'dotnet lambda deploy-function'. I had some issues with it (especially when changing the architecture to arm64), so I stopped looking into it. But to deploy you use:

```
dotnet lambda deploy-function aws-lambda-webapi --function-architecture arm64
```

And you can then test it like this:

```
dotnet lambda invoke-function aws-lambda-webapi -p "Good morning Jeroen"
```

# Useful reading

https://aws.amazon.com/blogs/compute/introducing-the-net-6-runtime-for-aws-lambda/

https://dev.to/lambdasharp/benchmarking-net-json-serializers-on-aws-lambda-279m

https://nodogmablog.bryanhogan.net/2022/04/net-6-on-aws-lambda-quick-demos/

https://nodogmablog.bryanhogan.net/2022/08/dotnet-7-custom-runtime-for-aws-lambda/  
https://nodogmablog.bryanhogan.net/2022/02/net-6-web-api-on-lambda-with-custom-a-runtime/  
https://github.com/normj/LambdaNETCoreSamples  
https://clearmeasure.com/hosting-dot-net-6-minimal-api-aws-lambda/

https://github.com/LambdaSharp/LambdaSharp.Benchmark