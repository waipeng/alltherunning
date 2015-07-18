---
layout: post
title:  Benchmarking NeCTAR Cloud
categories: NeCTAR
---
NeCTAR is made up of 8 nodes around Australia. Each node has quite a bit of discretion in choosing hardware and technologies to support their implementation of OpenStack. For example, Pawsey and Tasmania are both using Ceph as their cinder backend, whereas Melbourne is using a NetApp cluster for the same. As a result, it can be expected that performance will vary across the different node.

I was quite interested to figure out what the differences across the node are, and finally got down to benchmarking the cloud tonight. The benchmarks were done as follow:

1. Boot a m2.small instance
2. Install and kickstart a fio benchmark using Ansible
3. Upload the benchmark to a central machine using scp

Here are the results:

<iframe width="600" height="371" seamless frameborder="0" scrolling="no" src="https://docs.google.com/spreadsheets/d/1YEKURZ4KEsNhpsJKZWbuy1uwJ-evrMQ30a9A3Tzs88w/pubchart?oid=404716992&amp;format=interactive"></iframe>

* *Monash is doing iops throttling (40 iops)*

Overall, I'm pretty pleased with what was achieved in one night. Also, I managed to learn Ansible, after putting it off for so long!
