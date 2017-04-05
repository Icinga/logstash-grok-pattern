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

## Examples
The following examples demonstrate how the patterns can be used to parse Icinga
log files. These are just examples, you are free to use the patterns in which
way you want.

### Icinga 2 Main Log
The main log of Icinga 2 includes general information about the behaviour of the
process, each component of it and all enabled features. Logs are split into
three fields: `severity`,`facility` and `message`:

This example is based on a Logstash file input plugin. Other inputs can be used
as well. The date filter moves the timestamp of the log event to the field
`@timestamp`, which is used by default in Kibana to sort events.

```ruby
input {
  file {
    path => "/var/log/icinga2/icinga2.log"
    type => "icinga.main"
    sincedb_path   => "/dev/null"
    start_position => "beginning"
    codec => multiline {
      pattern             => "^\["
      negate              => true
      what                => previous
      auto_flush_interval => 2
    }
  }
}

filter {
  if [type] == "icinga.main" {
    grok {
      patterns_dir   => ["/etc/logstash/patterns/icinga"]
      match          => ["message", "%{ICINGA_MAIN}"]
      remove_field   => ["message"]
      add_tag        => ["filter.grok.icinga.main"]
      tag_on_failure => ["_grokparsefailure", "filter.grok.icinga.main.failure"]
    }

    date {
      match        => ["icinga.main.timestamp", "yyyy-MM-dd HH:mm:ss Z"]
      target       => "@timestamp"
      remove_field => ["icinga.main.timestamp"]
    }
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}
```
