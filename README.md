# Logstash Grok Pattern for Icinga

[Logstash](https://www.elastic.co/products/logstash) is a data processing
pipeline that processes data. It can receive, collect, parse, transform and
forward log events. This repository includes various
pattern for the Logstash filter [grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).

The grok filter is included in a default Logstash installation. It is used to
parse log events and split messages into multiple fields. Instead of writing
regular expressions, users use predefined patterns to parse logs. Besides the
included patterns, custom patterns can be added to extend the functionality.

1. [Installation](#installation)
2. [Examples](#examples)
    * [Icinga 2 Main Log](#icinga-2-main-log)
    * [Icinga 2 Debug Log](#icinga-2-debug-log)
    * [Icinga 2 Startup Log](#icinga-2-startup-log)

## Installation
Custom patterns need to be accessible by the Logstash daemon. It does not matter
where you put the files on the file system, as long as they are readable to the
`logstash` user.

```shell
mkdir /etc/logstash/patterns
cd /etc/logstash/patterns
git clone https://github.com/Icinga/logstash-grok-pattern.git icinga
```

To use custom patterns, include the directory with the `patterns_dir` paramter
in your grok filter:

```ruby
grok {
  patterns_dir   => ["/etc/logstash/patterns/icinga"]
  ...
}
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
`@timestamp`, which is used by default in
[Kibana](https://www.elastic.co/products/kibana) to sort events.

```ruby
input {
  file {
    path => "/var/log/icinga2/icinga2.log"
    type => "icinga.main"
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
      tag_on_failure => ["_grokparsefailure", "filter.icinga.main.grok.failure"]
    }

    date {
      match          => ["icinga.main.timestamp", "yyyy-MM-dd HH:mm:ss Z"]
      target         => "@timestamp"
      remove_field   => ["icinga.main.timestamp"]
      tag_on_failure => ["_dateparsefailur", "filter.icinga.debug.date.failure"]
    }
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}
```

### Icinga 2 Debug Log
The main log of Icinga 2 includes very detailed information about the behaviour
of the process, each component of it and all enabled features. Logs are split
into three fields: `severity`,`facility` and `message`:

This example is based on a Logstash file input plugin. Other inputs can be used
as well. The date filter moves the timestamp of the log event to the field
`@timestamp`, which is used by default in
[Kibana](https://www.elastic.co/products/kibana) to sort events.

```ruby
input {
  file {
    path => "/var/log/icinga2/debug.log"
    type => "icinga.debug"
    codec => multiline {
      pattern             => "^\["
      negate              => true
      what                => previous
      auto_flush_interval => 2
    }
  }
}

filter {
  if [type] == "icinga.debug" {
    grok {
      patterns_dir   => ["/etc/logstash/patterns/icinga"]
      match          => ["message", "%{ICINGA_DEBUG}"]
      remove_field   => ["message"]
      add_tag        => ["filter.grok.icinga.debug"]
      tag_on_failure => ["_grokparsefailure", "filter.icinga.debug.grok.failure"]
    }

    date {
      match          => ["icinga.debug.timestamp", "yyyy-MM-dd HH:mm:ss Z"]
      target         => "@timestamp"
      remove_field   => ["icinga.debug.timestamp"]
      tag_on_failure => ["_dateparsefailur", "filter.icinga.debug.date.failure"]
    }
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}
```

### Icinga 2 Startup Log
The startup log of Icinga 2 is generated each time when the daemon is restarted
or reloaded. It includes information about the amount of objects on startup,
which features are enabled, connection to the database and suchlike. The startup
log does not include a timestamp, the file is rewritten completely every time.

This example is based on a Logstash file input plugin. Other inputs can be used
as well.

```shell
input {
  file {
    path           => "/var/log/icinga2/startup.log"
    type           => "icinga.startup"
    start_position => "beginning"
    sincedb_path   => "/dev/null"
    codec          => multiline {
      pattern             => "^[a-z]*\/[a-zA-Z]*:"
      negate              => true
      what                => previous
      auto_flush_interval => 2
    }
  }
}

filter {
  if [type] == "icinga.startup" {
    grok {
      patterns_dir   => ["/etc/logstash/patterns/icinga"]
      match          => ["message", "%{ICINGA_STARTUP}"]
      remove_field   => ["message"]
      add_tag        => ["filter.grok.icinga.startup"]
      tag_on_failure => ["_grokparsefailure", "filter.grok.icinga.startup.failure"]
    }
  }
}

output {
  stdout {
    codec => "rubydebug"
  }
}
```
