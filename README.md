Overview
--------
[![Build Status](https://travis-ci.org/freeformz/logglier.png)](https://travis-ci.org/freeformz/logglier)

Send logged messages to Loggly using either the HTTP API or Syslog/UDP.

Can be used in place of Ruby's Logger
(<http://www.ruby-doc.org/stdlib/libdoc/logger/rdoc/>)

In fact, it (currently) returns an instance of Logger.

Usage
-----

    require 'logglier'

    log = Logglier.new(<INPUT URL>)

    log.info("hello from logglier")


### With Rails

config/environments/production.rb

    RailsApplication::Application.configure do
      config.logger = Logglier.new(<INPUT URL>)
    end


### With Rails 4

config/initializers/loggly.rb

    loggly = Logglier.new('https://logs-01.loggly.com/inputs/[your-customer-token]/tag/[your-rails-app]/', threaded: true, format: :json)
    Rails.logger.extend(ActiveSupport::Logger.broadcast(loggly))

^ Submitted by: https://github.com/cap10morgan


Input URLs
-------

### HTTP Inputs
    Logglier.new('https://logs.loggly.com/inputs/<id>')

The id is provided by loggly, look at the input's details page To make
sure the http client doesn't block too long read_timeout and
open_timeout are set to 2 seconds by default. This can be overridden
like so:

    Logglier.new('https://logs.loggly.com/inputs/<id>',
                 :read_timeout => <#>,
                 :open_timeout => <#> )

#### Threaded Delivery

Creating a new Logglier instance, pointed at a http input, with the
`:threaded => true` option will tell Logglier to deliver log messages
for that logger in a seperate thread. Each new Logglier instance gets
it's own delivery thread and those threads are joined at exit to ensure
log message delivery.

Example:

    Logglier.new('https://logs.loggly.com/inputs/<id>',
                 :threaded => true)

#### JSON Formatting

add the ':format => :json' when creating a new Logglier instance. Make
sure to use a HTTP input that has JSON enabled. Can also be used with
threaded delivery.

Example:

    Logglier.new('https://logs.loggly.com/inputs/<id>',
                 :format => :json)

Logglier uses [MultiJson](https://github.com/intridea/multi_json) to delegate the choice of JSON libraries to you, but I recommend using
[Yajl](https://github.com/brianmario/yajl-ruby), just require the json gem of your choice before logglier.

### Syslog TCP/UDP Inputs

    Logglier.new('[udp|tcp]://<hostname>:<port>/<facility>')

The facility is optional and defaults to 16 (local0) if none is
specified. Facilities are just integers from 0 to 23, see
<http://www.faqs.org/rfcs/rfc3164.html>

Logging
-------

Logglier.new returns a ruby Logger object, so take a look at:

http://www.ruby-doc.org/stdlib/libdoc/logger/rdoc/

The Logger's logdev has some special format handling though.

### Logging a string

    log.warn "test"

Will produce the following log message in Loggly:

    "<Date> severity=WARN, test"

### Logging a Hash

    log.warn :boom => :box, :bar => :soap

Will produce the following log message in Loggly:

    "<Date> severity=WARN, boom=box, bar=soap"

Bugs
-----

https://github.com/freeformz/logglier/issues

Pull requests welcome

TODO
-----

* Support ActiveSupport Notifications for newer rails
* Alternative https implementations (Typheous, Excon, etc). May be
  faster?
* EM Integration?
