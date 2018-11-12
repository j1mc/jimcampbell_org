+++
title = "Scale Your Auditing Events"
date = 2018-11-11T14:45:42-06:00
tags = ["audit", "tldw",]
categories = "sysadmin"
comments = false

[[resources]]
  name = "featured_image"
  src = "featured_image.jpg"
+++

The post is a summary of a talk, and is part of my
[Too Long, Didn't Watch](/tags/tldw) series of recorded talk summaries. The
talk itself is by [Philipp Krenn](https://xeraa.net/en/) and is taken from the
2018 [All Systems Go](https://cfp.all-systems-go.io/en/ASG2018/public/schedule)
conference. A video of the talk is
[available on YouTube](https://www.youtube.com/watch?v=MZVJuBitsmI).

For the technically-inclined, this presentation gives a higher-level overview
of the Elastic Stack and how it enhances core Linux auditing tools. If you want
a low-level, nitty-gritty talk on setting up auditing, you will not find it here.
However, if you have not-previously been as aware of `auditd` or the Elastic
Stack and its related tooling, I think you'll find this to be an informative
and helpful presentation.

### Starting with auditd

Discussions of auditing on Linux start with a look at a Linux auditing tool,
`auditd`.

Broadly speaking, `auditd` is the userspace component to the Linux auditing
system. It's responsible for writing audit records to the disk, and it
monitors (for example) file and network access, system calls, commands run by
a user, and other security events. These events can be grouped under three
main audit-related categories:

- User information
- Task information
- Processes / programs that are exiting

You can create an exclude rule to exclude certain events, but otherwise the
events will be logged by the auditing daemon.

### Viewing audit events

Linux provides utilities to view and parse the raw logs produced by `auditd`.
For starters, you can view the resulting logs via the `ausearch` or `aureport`
utilities. Running `sudo aureport` will give you a summary report of data
from /var/log/audit/audit.log. 

For more information on core Linux auditing utilities (and their resulting log
files), the presenter suggests taking a look at the
[related Red Hat documentation](https://access.redhat.com/documentation/en-US/red_hat_enterprise_linux/7/html/security_guide/sec-understanding_audit_log_files)
and also reviewing a collected set of 
[auditd audit rules](https://github.com/linux-audit/audit-userspace/tree/master/rules).
For those interested in auditing containers, the presenter notes that the
auditing of data from namespaces is still a
[work-in-progress](https://github.com/linux-audit/audit-kernel/issues/32#issuecomment-395052938).

### Elastic stack - Pulling it together

Once you have more than one system, you want to centralize data --- figure
out what events are happening and where those events are happening. The
presenter suggests that that is where Elasticsearch, Logstash and Kibana come
in. The presenter talks a bit about what was referred to as the ELK or ELK-Bee
stack (what is now just called the Elastic Stack), noting these components:

- Elasticsearch (where the data is stored)
- Logstash (One method of storing data)
- Kibana (the UI that you interact with to analyze logs)
- Beats (Used to actually collect the data)

This stack makes is possible to aggregate audit log data from different
systems. The presenter notes that this approach:

- Uses the `auditd` syntax so you can reuse what auditing rules you have.
- Correlates related events immediately
- Resolves UIDs to user names
- Pushes recorded events to Elasticsearch directly

### An example

As a demonstration, the presenter takes a look at a 'beat' configuration at
`/etc/auditbeat/auditbeat.yml`. He steps through
[examples of different auditing events](https://youtu.be/MZVJuBitsmI?t=611) that
you can configure the `auditbeat` module to track. The examples include:

- A set of 'identity' rules that log everything having to do with /etc/group,
  /etc/passwd, /etc/gshadow, and /etc/security/opasswd
- Logging read access to password of a 'developer' account
- Logging permission errors
- Logging processes that the 'connect' system call for IPv4 and IPv6
- When a user abuses their sudo privileges to read someone else's home
  directory
- All executed processes
- Logging anything that is elevating privileges

These are all useful events to audit. He also notes how you can check the file
integrity of certain files or folders. In particular, you can:

- Identify paths to monitor
- Compute a hash of files that are being monitored
- See which user had monitored files

As further demonstration, he shows a failed login attempt, and then views the
attempted login-related event logs in Kibana. He's able to see:

- Where failed attempts are coming from
- What user names they're using when trying to log in
- An example of a 'tag cloud' of failed attempts
- What processes users have restarted, what actions users have taken, etc.
- How you can filter down to different event types via tags
- How to verify file integrity via a hash

For further examples, there is a 
[auditbeat-in-action repository](https://github.com/xeraa/auditbeat-in-action)
on his github page, and you can try out
[his sample Kibana instance](https://dashboard.xeraa.wtf), as well.

Another part of his talk discusses a bit about why they didn't use eBPF. The
short answer is that eBPF requires newer kernels. On the other hand, `auditd`
is available on older kernels (think Centos 6).
