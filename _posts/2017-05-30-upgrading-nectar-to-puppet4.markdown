---
layout: post
title:  Upgrading NeCTAR to Puppet 4
categories: nectar
---

We've been running Puppet 3 for the longest time. With [Puppet 3 at
EOL](https://puppet.com/misc/puppet-enterprise-lifecycle), and [OpenStack going
to Puppet
4](https://openstack.nimeyo.com/100236/openstack-dev-puppet-fuel-packstack-tripleo-puppet-end-life)
for Octa/Pike, we really had little choice but to make ourselves Puppet 4 ready
too. After a couple of months on this project, we are (finally!) seeing light at
the end of the tunnel. It wasn't too difficult, more of grunt work really.
Here's documenting how we did it; hopefully it'll help someone needing to do the
same. Notes are to be used in tandem with PuppetLabs [official
guide](https://docs.puppet.com/puppet/4.9/upgrade_major_server.html).

# Get future parser ready
For this we had a script that runs future parser [^1] over all our manifests and
modules, and output the errors to a file that gets uploaded to Swift. Operators
from each site can take a look at the file (through Swift, so https) and figure
out which manifests needs to be updated. This script runs nightly, and as
changes get merged to production the output file has less and less entries.

An example of the error we see is:

```
Warning: Deprecation notice: Node inheritance is not supported in Puppet >=
4.0.0. See http://links.puppetlabs.com/puppet-node-inheritance-deprecation
   (at /etc/puppet/environments/myenv/manifests/nova.pp:6)
```

A minor thing to note is that we run validate twice, once with current parser
and once with future parser, as there are errors caught by one and not the
other.

```
for file in $FILES; do
    echo "Checking $file"
    puppet parser validate --config $DIR/puppet.conf "$file"
    puppet parser validate --config $DIR/puppet.conf --parser=future "$file" 
done
```

puppet.conf is just a 'fake' config with storeconfigs enabled

```
[main]
storeconfigs = true
```

# Check for 'Unacceptable Class'
In Puppet4 classnames can not have a `-` in them, so we needed to rename all our
classes. We did it without breaking current manifests by creating a new class
replacing `-` with `_`, and having the old class include the new.

For example, the old class was something like:

```
class ceilometer::agent-central inherits ceilometer {
    <code here>
 }
```

We create a new class

```
class ceilometer::agent_central inherits ceilometer {
    <code here>
 }
```

and rewrite the old to include the new. We also throw in a notify to bug the
operators to fix their manifests :)

```
class ceilometer::agent-central inherits ceilometer {
  include ::ceilometer::agent_central

  notify {'class ceilometer::agent-central is deprecated. Please use ceilometer::agent_central': }
}
```

Other than the nagging we also have a script that scans for all the classnames
that are no longer accepted in Puppet4

```
for class in $UNACCEPTABLE_CLASS`; do
    curl -s -G 'http://puppetdb.example.com:8080/v4/resources' --data-urlencode 'query=["and", ["=", "type", "Class"], ["~", "title", "'$class'"]]' | jq -r '.[] | select(length > 0) | select(.type == "Class") | [.title, .certname] | @csv'
done
```

which gives us a list of nodes that are using old classes

```
["compute01.example.com", "ceilometer::agent-central"]
["compute02.example.com", "ceilometer::agent-central"]
```

This output is sent to Swift for the operators to check.

# Stringify facts
We create a hiera value that sets `stringify_facts=false` in `puppet.conf`.
Operators from each site can set this hiera value on a compute node, a group of
nodes, or site-wide. The idea is that each operator can start off by setting this
value on some nodes, checking for errors in puppet runs, and then gradually set
them on more and more nodes.

# Catalog diffs against Puppetserver 3 (future parser)
A very useful tool that we have found is `octocatalog-diff` [^2], which allows us to
compile two catalogs and diff them against each other. 

For this stage we do the following:
1. Grab a list of active nodes from puppetdb

    ```
    curl -s -G 'http://puppetdb.example.com:8080/v4/nodes' | jq -r '.[] | [.certname] | @csv' | tr -d '"'
    ```

1. For each node, compile catalog using (1) current puppet master, and (2) a
   puppet3 server with future parser turned on, and compare the difference

    ```
    for node in $nodes; do
        octocatalog-diff --from-puppet-master puppet.example.com --to-puppet-master puppet3.example.com --puppet-master-api-version 2 -f production -t production --no-display-datatype-changes --no-validate-references --filter ArraySingleValue --filter StringInt -n $node    
    done
    ```

1. Upload all the diffs to Swift
1. Operators look at the diffs and fixes up the code
1. Our aim is to get 0 diffs between current production puppet master and a puppet3
  server with future parser

Sidenote: In the above command, `ArraySingleValue` and `StringInt` filters are
custom filters we wrote, to filter out some pedantic differences. Your code
might have lots of such minor differences; octocatalog-diff supports custom
filters to ignore them.

# Catalogs diffs against Puppetserver 4
This is similar to the previous, except that we check for differences between
puppet master and a puppet4 server.

```
for node in $nodes; do
    octocatalog-diff --from-puppet-master puppet.example.com --to-puppet-master puppet4.example.com --puppet-master-api-version 2 -f production -t production --no-display-datatype-changes --no-validate-references --filter ArraySingleValue --filter StringInt -n $node    
done
```

Similarly, these diffs gets upload to swift so that operators can see the errors
and fix them.

# Switchover servers
To decrease downtime, we set up a cluster of Puppet 4 servers, while keeping the
Puppet 3 servers running. To control which server a client connects to, an
operator can set a hiera variable changing `/etc/puppet/puppet.conf` on a puppet
slave, from
```
server = puppet.example.com
```
to
```
server = puppet4.example.com
```

This hiera can be changed at node level, nodegroup level or site wide, allowing
an operator to do a rolling upgrade.

[^1]: [Future parser](https://docs.puppet.com/puppet/3.8/experiments_future.html)
[^2]: [Octocatalog-diff](https://github.com/github/octocatalog-diff)
