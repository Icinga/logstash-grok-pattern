# Logstash Grok Pattern for Icinga

[Logstash](https://www.elastic.co/products/logstash) is a data processing
pipeline that processes data. It can receive, collect, parse, transform and
forward log events. This repository includes various
pattern for the Logstash filter [grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).

The grok filter is included in a default Logstash installation. It is used to
parse log events and split messages into multiple fields. Instead of writing
regular expressions, users use predefined patterns to parse logs. Besides the
included patterns, custom patterns can be added to extend the functionality.

## Installation
Custom patterns need to be accessible by the Logstash daemon. It does not matter
where you put the files on the file system, as long as they are readable to the
`logstash` user.

```shell
mkdir /etc/logstash/patterns
cd /etc/logstash/patterns
git clone https://github.com/Icinga/logstash-grok-pattern.git icinga
```
