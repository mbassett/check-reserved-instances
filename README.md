check-reserved-instances
============================

Check Reserved Instances - Compare instance reservations with running instances

Inspired by [epheph/ec2-check-reserved-instances](https://github.com/epheph/ec2-check-reserved-instances)

Amazon's reserved instances are a great way to save money when using EC2, RDS, ElastiCache, etc. An instance reservation is specified by an availability zone, instance type, and quantity. Correlating the reservations you currently have active with your running instances is a manual, time-consuming, and error prone process.

This quick little Python script uses boto to inspect your reserved instances and running instances to determine if you currently have any reserved instances which are not being used. Additionally, it will give you a list of non-reserved instances which could benefit from additional reserved instance allocations. The report may also be sent via email.


## Installation

Install the package using pip:

```
$ pip install check_reserved_instances
```


## Configuration

A sample configuration file is provided for easy use. By default, the script loads the configuration from config.ini alongside the check-reserved-instances executable.

```
$ cp config.ini.sample config.ini
```

### Configuring AWS Accounts/Credentials

Multiple AWS accounts are supported! Specify one or many sections with name `[AWS <name here>]`. These are the lists of AWS credentials that will used to query for instances. Replace `<name here>` with a nickname that want to provide in the report.

The following configuration options are supported:

- **aws_access_key_id** (Required str): The AWS IAM access key for a specific user.
- **aws_secret_access_key** (Required str): The AWS IAM secret key for a specific user.
- **region** (Optional str): The AWS region to query for the account. Defaults to us-east-1.
- **rds** (Optional bool): Boolean for whether or not to check RDS reserved instances.
- **elasticache** (Optional bool): Whether or not to check ElastiCache reserved instances.


### Email Report

The report can be sent via email (SMTP). Specify a section with name `[Email]`.

The following configuration options are supported:

- **smtp_host** (Required str): The hostname of the SMTP server.
- **smtp_port** (Optional int): The port the server uses for SMTP. Defaults to 25.
- **smtp_user** (Optional str): If your SMTP server requires authentication, specify a username. Defaults to None (no authentication).
- **smtp_password** (Optional str): If your SMTP server requires authentication, specify a password. Defaults to None (no authentication).
- **smtp_recipients** (Required str): The email addresses to send the email alert to. Specify one or many email addresses delimited by comma.
    - Example:
        - smtp_recipients = user1@company.com
        - smtp_recipients = user1@company.com, user2@company.com
- **smtp_sendas** (Optional str): The email address to send the emails as. Defaults to `root@localhost`.
- **smtp_tls** (Optional bool): Whether or not the SMTP server should use TLS to connect. Defaults to False.


## Usage

The following optional parameter is supported:

* -c, --config : Specify a custom path to the configuration file.

Ideally, this script should be ran in a cronjob:

```
# Run on the first day of every month
0 0 1 * * check-reserved-instances --config config.ini
```

For one-time use, execute the script:

```
$ check-reserved-instances --config config.ini
AWS account1 Reserved Instances Report
###############################################

Below is the report on EC2 reserved instances:


UNUSED RESERVATION! (1) m1.small    us-east-1b

UNUSED RESERVATION! (1) m2.2xlarge  us-east-1a


NOT RESERVED!  (1) t1.micro    us-east-1c

NOT RESERVED!  (2) m1.small    us-east-1d

NOT RESERVED!  (3) m1.medium   us-east-1d

NOT RESERVED!  (1) m2.2xlarge  us-east-1b


(23) running on-demand EC2 instances
(18) EC2 reservations
###############################################

Not sending email for this report
```

In this example, you can easily see that an m2.2xlarge was spun up in the wrong AZ (us-east-1b vs. us-east-1a), as well as an m1.small. The "Instance not reserved" section shows that you could benefit from reserving:
* (1) t1.micro
* (1) m1.small (not 2, since you'll likely want to move your us-east-1b small to us-east-1d)
* (3) m1.medium


## Required Permissions

The following example policy is the minimum set of permissions needed to run the reporter:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeReservedInstances",
                "rds:DescribeDBInstances",
                "rds:DescribeReservedDBInstances",
                "elasticache:DescribeCacheClusters",
                "elasticache:DescribeReservedCacheNodes"
            ],
            "Resource": "*"
        }
    ]
}
```


## TODO

- Overhaul format of report (one table with all accounts/services?)
- In report, add
    - time since launch with each instance in the NOT RESERVED
    - cost-savings of each UNUSED RESERVATION instance type
- Move templates to package data and install to operating system folder (ex. /etc/check-reserved-instances) for easy editing


## Contributing

Bug reports and pull requests are welcome. If you would like to contribute, please create a pull request against master. Include unit tests if necessary, and ensure that your code passes all linters (see tox.ini).