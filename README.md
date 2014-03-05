# Hiera AWS Backend

[![Build Status](https://travis-ci.org/Jimdo/hiera-aws.png?branch=master)](https://travis-ci.org/Jimdo/hiera-aws)

This backend for [Hiera] allows you to retrieve information from AWS that you
can use in your Puppet code at runtime. For example, you can ask the backend to
get a list of all nodes part of a specific ElastiCache cluster.

This project was inspired by the [hiera-cloudformation] backend.

## Installation

You can install the gem this way:

    $ gem install hiera-aws

## Usage

First, add the backend to the list of backends in `hiera.yaml`:

```yaml
---
:backends:
  - yaml
  - aws
```

Next, add the AWS services supported by this backend to the hierarchy:

```yaml
:hierarchy:
  - aws/elasticache
  - aws/rds
```

The following AWS privileges are required for Hiera to work:

- `AmazonEC2ReadOnlyAccess`
- `AmazonElastiCacheReadOnlyAccess`
- `AmazonRDSReadOnlyAccess`
- `AWSCloudFormationReadOnlyAccess`
- `IAMReadOnlyAccess`

To grant those privileges, you either have to assign the EC2 instances an IAM
role (preferred) or provide credentials for a user with the same privileges via
the backend configuration in `hiera.yml`:

```yaml
:aws:
  :access_key_id: your_aws_access_key_id_here
  :secret_access_key: your_aws_secret_access_key_here
```

## Hiera Keys

The backend currently supports the following keys that you can pass to the
`hiera()` function to look up objects in AWS.

### cache_nodes_by_cache_cluster_id

Returns an array of all nodes part of a ElastiCache cluster. The cluster is
identified by its physical ID which must be passed to the backend via the Puppet
fact `$cache_cluster_id`. The returned array has the format `["host1:port",
"host2:port"]`.

Usage:

```
$cache_cluster_id = "your_cluster_id"

cluster_nodes = hiera("cache_nodes_by_cache_cluster_id")
```

### redis_cluster_nodes_for_cfn_stack

Returns an array of all Redis cluster nodes for the CloudFormation stack of an
EC2 instance. The instance is identified by the Puppet fact `$ec2_instance_id`.
The returned array has the format `["host1", "host2"]`.

Usage:

```
cluster_nodes = hiera("redis_cluster_nodes_for_cfn_stack")
```

### memcached_cluster_nodes_for_cfn_stack

Returns an array of all Memcached cluster nodes for the CloudFormation stack of
an EC2 instance. The instance is identified by the Puppet fact
`$ec2_instance_id`. The returned array has the format `["host1", "host2"]`.

Usage:

```
cluster_nodes = hiera("memcached_cluster_nodes_for_cfn_stack")
```

### rds tag=value...

Returns an array of all RDS database instances that have one or more tags
assigned.

For each instance in the array the following hash is returned:

```json
{
    "db_instance_identifier" => "some-instance-identifier",
    "endpoint" => {"address" => "some.rds.endpoint", "port" => 3306},
    "engine" => "mysql"
}
```

Usage:

```
# Get all database instances
rds_instances = hiera("rds")

# Get all database instances that have a tag named "environment" with the value "dev"
rds_instances = hiera("rds environment=dev")

# Get all database instances that have two specific tags
rds_instances = hiera("rds environment=production role=mgmt-db")

# Accessing specific properties of the first database instance
$instance_identifier = $rds_instances[0]['db_instance_identifier']
$endpoint_address = $rds_instances[0]['endpoint']['address']
```

## License and Authors

* Author:: Mathias Lafeldt (mathias.lafeldt@jimdo.com)
* Author:: Deniz Adrian (deniz.adrian@jimdo.com)

Copyright:: 2013, Jimdo GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Contributing

We welcome contributed improvements and bug fixes via the usual workflow:

1. Fork this repository
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new pull request


[Hiera]: http://docs.puppetlabs.com/hiera/1/puppet.html
[hiera-cloudformation]: https://github.com/fanduel/hiera-cloudformation
