<!-- prettier-ignore -->
!!! warning
    This datasource is experimental.
    Its syntax and behavior may change at any time!

This datasource returns the latest [Amazon Machine Image](https://docs.aws.amazon.com/en_en/AWSEC2/latest/UserGuide/AMIs.html) via the AWS API (valid credentials required).

Because there is no general `packageName`, you have to use the [describe images filter](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-ec2/interfaces/describeimagescommandinput.html#filters) as minified JSON as a `packageName`.

Example:

```yaml
# Getting the latest official EKS image from AWS (account '602401143452' for eu-central-1) for EKS 1.21 (name matches 'amazon-eks-node-1.21-*') would look as a describe images filter like:

[
  {
    "Name": "owner-id",
    "Values": [ "602401143452" ]
  },
  {
    "Name": "name",
    "Values": [ "amazon-eks-node-1.21-*" ]
  }
]

# In order to use it with this datasource, you have to minify it:

[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.21-*"]}]
```

At the moment, this datasource has no "manager".
You have to use the regex manager for this.

**Usage Example**

Here's an example of using the regex manager:

```javascript
module.exports = {
  regexManagers: [
    {
      fileMatch: ['.*'],
      matchStrings: [
        '.*amiFilter=(?<packageName>.*?)\\n(.*currentImageName=(?<currentDigest>.*?)\\n)?(.*\\n)?.*?(?<depName>[a-zA-Z0-9-_:]*)[ ]*?[:|=][ ]*?["|\']?(?<currentValue>ami-[a-z0-9]{17})["|\']?.*',
      ],
      datasourceTemplate: 'aws-machine-image',
      versioningTemplate: 'aws-machine-image',
    },
  ],
};
```

Or as JSON:

```yaml
{
  'regexManagers':
    [
      {
        'fileMatch': ['.*'],
        'matchStrings':
          [
            ".*amiFilter=(?<packageName>.*?)\\n(.*currentImageName=(?<currentDigest>.*?)\\n)?(.*\\n)?.*?(?<depName>[a-zA-Z0-9-_:]*)[ ]*?[:|=][ ]*?[\"|']?(?<currentValue>ami-[a-z0-9]{17})[\"|']?.*",
          ],
        'datasourceTemplate': 'aws-machine-image',
        'versioningTemplate': 'aws-machine-image',
      },
    ],
}
```

This would match every file, and would recognize the following lines:

```yaml
# With AMI name mentioned in the comments
# amiFilter=[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.21-*"]}]
# currentImageName=unknown
my_ami1: ami-02ce3d9008cab69cb
# Only AMI, no name mentioned
# amiFilter=[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.20-*"]}]
# currentImageName=unknown
my_ami2: ami-0083e9407e275acf2
```

```typescript
const myConfigObject = {
  // With AMI name mentioned in the comments
  // amiFilter=[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.21-*"]}]
  // currentImageName=unknown
  my_ami1: 'ami-02ce3d9008cab69cb',
};

/**
 * Only AMI, no AMI name mentioned
 * amiFilter=[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.20-*"]}]
 * currentImageName=unknown
 */
const my_ami2 = 'ami-0083e9407e275acf2';
```

```hcl
resource "aws_instance" "web" {

    # Only AMI, no name mentioned
    # amiFilter=[{"Name":"owner-id","Values":["602401143452"]},{"Name":"name","Values":["amazon-eks-node-1.20-*"]}]
    # currentImageName=unknown
    ami = "ami-0083e9407e275acf2"

    count = 2
    source_dest_check = false

    connection {
        user = "root"
    }
}
```