# aws-cloudwatch-agent-install-onprem
Install and configure Cloudwatch Agent on On-Premises server for logs and metrics reporting into AWS

# Download Cloudwatch Agent Installer Package
Find the appropriate package for your system. Remember that AMD64 and ARM64 are two different type of processors.
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html

For example on a Ubuntu Linux you can dowload using following command.
```wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb``` 

# Install Cloudwatch Agent
Follow above AWS guide to install the package you have downloaded. For example on Ubuntu server you install it like this.
```sudo dpkg -i -E ./amazon-cloudwatch-agent.deb```

# Configure AWS workspace and CloudWatch Profile
Configure AWS credentials and configuration with a user credentails that has sufficient access to communicate with AWS CloudWatch & SSM. The Profile name used in here is what you will create and use for this monitoring and it will publish metrics to AWS CloudWatch within that profile name. If it doesn't exist, don't worry, it will create a workspace in CloudWatch. You can do all sorts of things with AWS configure - `https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html`.

```sudo aws configure --profile AmazonCloudWatchAgent```
```
$ aws configure --profile AmazonCloudWatchAgent
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

# CloudWatch Configuration
## Create CloudWatch Configuration File using Wizard
Run the Wizard to create CloudWatch Configuration file. It will ask you series of questions and based on the response it will create a configuration file. You can also get Wizard to store the config file in AWS Parameter Store (but don't have to) for future use or further rollout. For more details refer to `https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html`.
You do not have to use CollectD or other Statistics/ Network performance daemons and will still get basic OS metrics.

```sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard```

This will create a configurairon file in JSON format which you can view and modify as required. in following location.
```/opt/aws/amazon-cloudwatch-agent/bin/config.json```
## Manually created CloudWatch Configuration file
You can manually create the configuraiton file that will then be used by CloudWatch agent to initiate the metric push. For more details please see ```https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html```.

A sample file `config.json` with some basic metrics is also included within this reposotory which you can modify and apply to suit your requirement. You can use following command to back up your old config file and copy the file in default cloudwatch installation directory.

```
sudo mv /opt/aws/amazon-cloudwatch-agent/bin/config.json /opt/aws/amazon-cloudwatch-agent/bin/config.json.$$.bak
sudo cp config.json /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

# Start the CloudWatch Agent
In this step you will start the agent. For more details see `https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-commandline-fleet.html`.
You should carefully watch the output for any errors as that will be important to resolve any issue experienced here.

```sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m onPremise -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json```

# Check the Agent status

```sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m onPremise -a status```


# Verifying the metrics in CloudWatch console.
Log into AWS Console and go to Cloudwatch.
Select the appropriate region.
Click on Metrics.
Select All Metrics and look for something like ``CWAgent`` and should see all lots of metrics there.

# Errors and troubleshooting
I had an error to do with CollectD file or directory not found. I could touch the locations and get the agent running but for it to work correctly I installed collectd separately using following command on Ubuntu server. I later removed all CollectD metrics and removed it from the config so do not need that anymore.

```sudo apt-get install collectd```

# Remove configuration and restart again
If for some reason you didn't like it or it didn't work, you can remove the configuraiton and start the process again. Before you start you should stop the agent and remove log and configuration files.

```
sudo systemctl stop amazon-cloudwatch-agent.service
sudo mv /opt/aws/amazon-cloudwatch-agent/bin/config.json /opt/aws/amazon-cloudwatch-agent/bin/config.json.$$.bak
sudo mv /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml.$$.bak
sudo mv /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log.$$.bak
sudo mv /opt/aws/amazon-cloudwatch-agent/logs/configuration-validation.log /opt/aws/amazon-cloudwatch-agent/logs/configuration-validation.log.$$.bak
```
Once removed you can start the wizard again to generate the config.

```sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard```
