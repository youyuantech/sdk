# macchina.io Remote Manager Device Agent (WebTunnelAgent)

In order to connect a device to macchina.io Remote Manager, a program (Remote Manager Device Agent
or `WebTunnelAgent`) needs to be installed on the device. For initial testing, the agent can also be
installed on a Windows, Linux or macOS machine in the same network as the device.

`WebTunnelAgent` is built as part of the [macchina.io Remote Manager SDK](../../README.md).

## Running WebTunnelAgent

`WebTunnelAgent` needs a configuration file named `WebTunnelAgent.properties` to work.
The easiest way to obtain the configuration file is to generate and download it from the
Remote Manager Account page. A [basic configuration file](WebTunnelAgent.properties)
is also available in this directory.

The configuration file should be located in the same directory as `WebTunnelAgent`.
Alternatively, the path to the configuration file can be specified when starting
`WebTunnelAgent` using the `--config-file` option:

```
WebTunnelAgent --config=/path/to/WebTunnelAgent.properties
```

Note that on Windows the command-line parameters must be given Windows-style:

```
WebTunnelAgent /config=c:\\path\to\WebTunnelAgent.properties
```

### Running WebTunnelAgent as Daemon

On Linux and macOS systems, `WebTunnelAgent` can be run as a daemon.
To do so, pass the `--daemon` option:

```
WebTunnelAgent --daemon
```

Additional options can be passed to set the daemon's umask, or create
a pidfile. Please run

```
WebTunnelAgent --help
```

for more information.


### Running WebTunnelAgent as Windows Service

`WebTunnelAgent` can be registered as a service on Windows.
To do so, start it with the `/registerService` option.
This must be done from a Command Prompt with Administrator privileges.

```
WebTunnelAgent /registerService
```

Additional options such as `/displayName`, `/description` and `/startup`
can also be used to specify service properties.
Please run

```
WebTunnelAgent /help
```

for more information.

The `WebTunnelAgent` service can be started from the *Services* MMC plugin, or via
the command-line:

```
net start WebTunnelAgent
```

To stop the `WebTunnelAgent` service, run:

```
net stop WebTunnelAgent
```

To unregister the `WebTunnelAgent` service, run:

```
WebTunnelAgent /unregisterService
```


## Configuring WebTunnelAgent

`WebTunnelAgent` is configured using a configuration file usually named
`WebTunnelAgent.properties`. If the file is located in the same directory
as the `WebTunnelAgent` executable, or a parent directory, it will be picked up
automatically. Otherwise, the path to the configuration file can be specified using
the `--config-file` or `/config-file` command-line parameter.


### Configuration File Format

The configuration file can be edited with any text editor. It
contains a number of settings that control various aspects of the
`WebTunnelAgent`. For better readability, the configuration
file is divided into different sections. The sections and settings
are described in the following.

The configuration file format is as follows. A line starting with a hash '#' or
exclamation mark '!' is treated as a comment and is ignored. Every other line denotes a
setting definition in the form

```
key = value
```

or

```
key: value
```

Keys and values may contain special characters represented by the following escape sequences:

  * `\t`: tab (0x09)
  * `\n`: line feed (0x0a)
  * `\r`: carriage return (0x0d)
  * `\f`: form feed (0x0c)

For every other sequence that starts with a backslash, the backslash is removed.
Therefore, the sequence `\a` would just yield an `'a'`.
A value can spread across multiple lines if the last character in a line
(the character immediately before the carriage return or line feed character) is a
single backslash. Setting keys are case sensitive. Leading and trailing whitespace is
removed from both keys and values. A setting name can neither contain a colon `':'`
nor an equal sign `'='` character.
In a value it’s possible to reference another value using the syntax `${key}`.
There are a number of system settings that can be used as well:

  * `system.osName`: the operating system name
  * `system.osVersion`: the operating system version
  * `system.osArchitecture`: the operating system architecture
  * `system.nodeName`: the node (or host) name
  * `system.nodeId`: system ID, based on the Ethernet address (format `"xxxxxxxxxxxx"`) of the
    first Ethernet adapter found on the system
  * `system.currentDir`: the current working directory
  * `system.homeDir`: the user's home directory
  * `system.tempDir`: the system's temporary directory
  * `system.dateTime`: the current UTC date and time, formatted in ISO 8601 format (example: `2005-01-01T11:00:00Z`)
  * `system.pid`: the current process ID
  * `system.env.NAME`: the value of the environment variable with the given NAME
  * `application.path`: the absolute path to the `WebTunnelAgent` executable
  * `application.name`: the name of the `WebTunnelAgent` executable
  * `application.dir`: the path to the directory containing the `WebTunnelAgent` executable

Settings not recognized by `WebTunnelAgent` will be ignored. However, it is still
possible to reference these setting’s values in other settings.
This can be used to introduce "macro" settings which are referenced in multiple settings.


### Primary Settings

#### webtunnel.domain

This setting specifies the domain UUID which is used to associate the device with a user account
or user group. It can also be used to group devices. The value should be an all lower-case
UUID like `d5f6e710-49c9-4e21-9cd7-2ae0054c3f13`. How domains are assigned to devices
and users depends on the specific Remote Manager server instance. For example, the
[public test server](https://reflector.my-devices.net) assigns a unique domain UUID to
each new registered user account.

Please note that it's not allowed to change the domain UUID of an already registered device
unless device authentication has been enabled.

#### webtunnel.deviceId

The device ID is used to uniquely address the device and therefore must be unique
for all devices on a Remote Manager server.
The ID must be a valid domain name, as it will be used as part of the device URL.
It may contain letters `a`-`z` and digits `0`-`9`, as well as dashes (`-`),
but must not begin with a dash.

It's possible to use the system's Ethernet address (`${system.nodeId}`) as ID
or part of the ID. However, please note that on some systems the Ethernet address
reported by `${system.nodeId}` may change between reboots if the system has
multiple network adapters.

#### webtunnel.deviceName

This optional property can be used to set the device `name` property shown in the
Remote Manager dashboard and device page.

Note that if enabled, this will cause the device name property to be
set every time `WebTunnelAgent` connects to the Remote Manager, therefore
overwriting any changes made in the Remote Manager's web interface, shell
or API.

You can specify a name, or use a configuration variable like
`${system.nodeName} `or refer to an environment variable like
`${system.env.HOSTNAME}`.

#### webtunnel.version

This optional property can be used to set the device `version` property shown
in the Remote Manager device page (and optionally dashboard, if configured).
It's intended to report the device's firmware version number or something equivalent.

Note that if enabled, this will cause the device `version` property to be
set every time `WebTunnelAgent` connects to the Remote Manager, therefore
overwriting any changes made in the Remote Manager's web interface, shell
or API.

#### webtunnel.userAgent

This optional property can be used to set the device `userAgent` property
shown in the Remote Manager device page.

#### webtunnel.host

This setting specifies the IP address or domain name of the target device.
If `WebTunnelAgent` is running directly on the target device, this can be
the loopback address `127.0.0.1`.

#### webtunnel.ports

This setting specifies a comma-separated list of port numbers to forward.
It should include the port number of the device's web server
(usually 80, but can be another one). Can also include
SSH (22), VNC (5900) or other TCP ports.

#### webtunnel.httpPort

This setting specifies the port number of the device's web server. Must only be
set if different from default HTTP port 80. Must also be included in the `webtunnel.ports` list.

#### webtunnel.vncPort

The optional setting specifies the port number of the device's VNC server.
Used to enable VNC support in the web interface.
Must also be included in the webtunnel.ports list.
If not set VNC access will not be enabled via the Remote Manager.

#### webtunnel.reflectorURI

The URL of the Remote Manager server.

#### webtunnel.username

The username of the device. Always consists of device ID (`webtunnel.deviceId`) and
the domain UUID (`webtunnel.domain`), separated by `'@'`.

Should always be set to:

```
webtunnel.username = ${webtunnel.deviceId}@${webtunnel.domain}
```

#### webtunnel.password

The device password, used for authenticating the device.
Device authentication is disabled on the public demo server, so can be left
empty. Device authentication can be enabled for private Remote Manager instances.

#### webtunnel.connectTimeout

The timeout (given in seconds) for connecting to the local (forwarded) server socket,
e.g., the device's web server.

#### webtunnel.localTimeout

The send and receive timeout (given in seconds) for local (forwarded) socket connections,
i.e., the connection to the device's web server.

#### webtunnel.remoteTimeout

The timeout (given in seconds) for the WebTunnel connection to the Remote Manager
server. If half of the timeout expires, the `WebTunnelAGent` will send a PING message to the
Remote Manager server. If the PING is not answered by the server, the `WebTunnelAgent`
will terminate the connection and attempt to re-connect.

#### webtunnel.status.notify

This optional setting specifies the path to an executable that is started whenever
the state of the tunnel connection to the Remote Manager changes. The current state
will be passed as command-line argument to the executable and will be one of:

  * `connected`: the tunnel connection has been established.
  * `disconnected`: the tunnel connection has been disconnected.
  * `error`: there has been an error establishing the tunnel. A second parameter will
    also be passed containing more information about the error.

#### webtunnel.threads

The number of I/O threads the `WebTunnelAgent` should use. Should be left at the default
(4).


### HTTP Configuration

#### http.timeout

The timeout (given in seconds) for the initial HTTP(S) connection to the
Remote Manager server.

#### http.proxy.enable

Enable (set to `true`) or disable (set to `false`) HTTP proxy support for the
HTTP/WebTunnel connection to the Remote Manager server.
If enabled, the proxy host, port, username and password should also be specified.

#### http.proxy.host

The domain name or IP address of the HTTP proxy server to use.

#### http.proxy.port

The port number of the HTTP proxy server to use.

#### http.proxy.username

The username for authenticating against the HTTP proxy server. Can be left empty
if no proxy authentication is required.

#### http.proxy.password

The password for authenticating against the HTTP proxy server. Can be left empty
if no proxy authentication is required.


### SSL/TLS Configuration

#### tls.acceptUnknownCertificate

Enable (`true`) or disable (`false`) accepting an unknown certificate from the
Remote Manager server. Should only be used for testing purposes, e.g., while using
a self-signed certificate. Should not be used in production setups.

#### tls.ciphers

This setting is used to specify a list of acceptable OpenSSL ciphers. Only used
if `WebTunnelAgent` has been built with OpenSSL.

#### tls.extendedCertificateVerification

Enable (`true`) or disable (`false`) extended (domain name) certificate validation.
If set to `true`, which is highly recommended, `WebTunnelAgent` will verify that the
common name (or one of the Subject Alternative names) of the server certificate
matches the server domain name, as specified in `webtunnel.reflectorURI`.

#### tls.caLocation

This setting specifies a directory or file containing trusted root certificates
for OpenSSL. Can be left empty to use the built-in default OpenSSL root certificates.
Please note that OpenSSL on Windows may not include a list of trusted root certificates.

#### tls.certificate

This optional setting specifies the path to an X509 certificate file in PEM format
used for authenticating the device against the server (together with a private key,
specified using tls.privateKey property).

#### tls.privateKey

This optional setting specifies the path to an X509 private key file in PEM format
used for authenticating the device against the server (together with a certificate,
specified using tls.certificate property).

### Logging

`WebTunnelAgent` supports a very flexible logging configuration, allowing fine-grained
control over the amount of logging data produced. Logging information can be written to the
console, to a log file, or to the syslog daemon.

Logging is based on the concept of *loggers*, *channels* and *formatters*.
A logger creates a log message, which is then formatted using a formatter, and
then sent to a channel. The kind of channel decides whether the message is
written to the console, to a file, or to the syslog daemon. Loggers form a hierarchy,
with a special logger, the *root logger*, at the base.
A logger inherits its configuration from the logger(s) above in the hierarchy.
If only the root logger is configured, all loggers will inherit the configuration of the
root logger. In the following, we will only configure the root logger, as
this is sufficient for most production purposes. A logger’s configuration consists of the
channel the logger is connected to, as well as its log level. Every log message is
tagged with a priority, or log level. A logger will only forward messages with a
priority higher than the configured log level.
The following priorities or log levels are available:

  * `none` (turns off logging)
  * `fatal` (highest priority)
  * `critical`
  * `error`
  * `warning`
  * `notice`
  * `information`
  * `debug`
  * `trace` (lowest priority)

#### logging.loggers.root.channel

This setting specifies which channel is connected to the root logger. A channel with the
name specified here must be defined in the configuration file, otherwise the application
won’t start up. The default configuration file specifies two channels, one
named `console` and one named `file`. Additional channels can be added if necessary.

#### logging.loggers.root.level

This setting specifies the log level of the root logger. Specify one of the log levels
`none`, `fatal`, `critical`, `error`, `warning`, `notice`, `information`, `debug` or `trace`.
The lower the level, the more information will be logged. For production purposes,
the log level `notice` or `warning` is recommended.

#### logging.channels.file.class

This setting creates a channel named `file` for writing to a log file (`FileChannel`).

#### logging.channels.file.pattern

This setting specifies the format of the log messages for the file channel.
See *Logging Format Placeholders* for a list of supported format placeholders.

#### logging.channels.file.path

The path of the log file.

#### logging.channels.file.rotation

The log file rotation strategy. Log files can be rotated based on size or time interval.
Rotating a log file means closing the current log file, renaming ("archiving") it,
and creating a new log file. The following values can be given:

  * `never`: No log rotation. The log file will grow indefinitely. Default.
  * `[day,][hh]:mm`: The log file is rotated on specified day/time.
    - `day` is specified as long or short day name (Monday/Mon, Tuesday/Tue, ... ) and can be
      omitted, in which case log is rotated daily.
    - `hh`: hour – valid hour range is 00-23; can be omitted, in which case log is rotated every hour.
    - `mm`: minute – valid minute range is 00-59 and must be given.
  * `daily`: the file is rotated daily.
  * `weekly`: The file is rotated every seven days.
  * `monthly`: The file is rotated every 30 days.
  * `<n> minutes`: The file is rotated every `<n>` minutes,  where `<n>` is an integer greater than zero.
  * `<n> hours`: The file is rotated every `<n>` hours, where `<n>` is an integer greater than zero.
  * `<n> days`: The file is rotated every `<n>` days, where `<n>` is an integer greater than zero.
  * `<n> weeks`: The file is rotated every `<n>` weeks, where `<n>` is an integer greater than zero.
  * `<n> months`: The file is rotated every `<n>` months, where `<n>` is an integer greater than zero and a month has 30 days.
  * `<n>`: The file is rotated when its size exceeds `<n>` bytes.
  * `<n> K`: The file is rotated when its size exceeds `<n>` Kilobytes.
  * `<n> M`: The file is rotated when its size exceeds `<n>` Megabytes.

NOTE: For periodic log file rotation (daily, weekly, monthly, etc.), the date and time of
log file creation or last rotation is written into the first line of the log file.
This is because there is no reliable way to find out the real creation date of a file on
many platforms (e.g., most Unix platforms do not provide the creation date, and Windows
has its own issues with its "File System Tunneling Capabilities").

#### logging.channels.file.time

Using this setting it is possible to specify whether the times used for rotation are in
UTC or local time. The following values are supported:

  * `utc`: Rotation is based on UTC time (default).
  * `local`: Rotation is based on local time.

#### logging.channels.file.archive

Using the this setting it is possible to specify how archived log files are named. The following values are supported:

  * `number`: A number, starting with 0, is appended to the name of archived log files.
    The newest archived log file always has the number 0. For example, if the log file is named
    `WebTunnelAgent.log`, and it fulfils the criteria for rotation, the file is renamed to
    `WebTunnelAgent.log.0`. If a file named `WebTunnelAgent.log.0` already exists, it is renamed to
    `WebTunnelAGent.log.1`, and so on.
  * `timestamp`:  A timestamp is appended to the log file name. For example, if the log file is named
    `WebTunnelAgent.log`, and it fulfils the criteria for rotation, the file is renamed to
    `WebTunnelAgent.log.20190510113000`.

#### logging.channels.file.purgeAge

Archived log files can be automatically purged, either if they reach a certain age, or if
the number of archived log files reaches a given maximum number.
This is controlled by the purgeAge and purgeCount settings.

The purgeAge property can have the following values:

  * `<n> [seconds]`: The maximum age is `<n>` seconds.
  * `<n> minutes`: The maximum age is `<n>` minutes.
  * `<n> hours`: The maximum age is `<n>` hours.
  * `<n> days`: The maximum age is `<n>` days.
  * `<n> weeks`: The maximum age is `<n>` weeks.
  * `<n> months`: The maximum age is `<n>` months, where a month has 30 days.

#### logging.channels.file.purgeCount

This setting specifies the maximum number of archived log files.
If the number is exceeded, archived log files deleted, starting with the oldest.

#### logging.channels.file.compress

Archived log files can be compressed using the *gzip* compression method. The following values are supported:

  * `true`: Compress archived log files.
  * `false`: Do not compress archived log files.

#### logging.channels.console.class

This setting creates a channel named `console` for writing to standard output (`ColorConsoleChannel`).

#### logging.channels.console.pattern

This setting specifies the format of the log messages for the `console` channel.
See *Logging Format Placeholders* for a list of supported format placeholders.

#### Logging Format Placeholders

  * `%s`: message source
  * `%t`: message text
  * `%l`: message priority level (1 .. 7)
  * `%p`: message priority (Fatal, Critical, Error, Warning, Notice, Information, Debug, Trace)
  * `%q`: abbreviated message priority (F, C, E, W, N, I, D, T)
  * `%P`: message process identifier
  * `%T`: message thread name
  * `%I`: message thread identifier (numeric)
  * `%N`: node or host name
  * `%U`: message source file path (empty string if not set)
  * `%u`: message source line number (0 if not set)
  * `%w`: message date/time abbreviated weekday (Mon, Tue, ...)
  * `%W`: message date/time full weekday (Monday, Tuesday, ...)
  * `%b`: message date/time abbreviated month (Jan, Feb, ...)
  * `%B`: message date/time full month (January, February, ...)
  * `%d`: message date/time zero-padded day of month (01 .. 31)
  * `%e`: message date/time day of month (1 .. 31)
  * `%f`: message date/time space-padded day of month ( 1 .. 31)
  * `%m`: message date/time zero-padded month (01 .. 12)
  * `%n`: message date/time month (1 .. 12)
  * `%o`: message date/time space-padded month ( 1 .. 12)
  * `%y`: message date/time year without century (70)
  * `%Y`: message date/time year with century (1970)
  * `%H`: message date/time hour (00 .. 23)
  * `%h`: message date/time hour (00 .. 12)
  * `%a`: message date/time am/pm
  * `%A`: message date/time AM/PM
  * `%M`: message date/time minute (00 .. 59)
  * `%S`: message date/time second (00 .. 59)
  * `%i`: message date/time millisecond (000 .. 999)
  * `%c`: message date/time centisecond (0 .. 9)
  * `%F`: message date/time fractional seconds/microseconds (000000: 999999)
  * `%z`: time zone differential in ISO 8601 format (Z or +NN.NN)
  * `%Z`: time zone differential in RFC format (GMT or +NNNN)
  * `%E`: epoch time (UTC, seconds since midnight, January 1, 1970)
  * `%L`: use local time zone for dates and times (must be specified before any of the date/time placeholders)
  * `%%`: percent sign

