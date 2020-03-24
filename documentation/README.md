# ![HAProxy](../assets/images/haproxy-weblogo-210x49.png "HAProxy")

## HAProxy kubernetes ingress controller

Options for starting controller can be found in [controller.md](controller.md)

### Available annotations

> :information_source: Ingress and service annotations can have `ingress.kubernetes.io`, `haproxy.org` and `haproxy.com` prefixes
>
> Example: `haproxy.com/ssl-redirect` and `haproxy.org/ssl-redirect` are same annotation

| Annotation | Type | Default | Dependencies | Config map | Ingress | Service |
| - |:-:|:-:|:-:|:-:|:-:|:-:|
| [blacklist](#access control) | [IPs or CIDRs](#access control) | "" |  |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [check](#backend-checks) | ["true", "false"] | "true" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [check-http](#backend-checks) | string |  | [check](#backend-checks) |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [check-interval](#backend-checks) | [time](#time) |  | [check](#backend-checks) |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [cookie-persistance](#cookie-persistance) | string | "" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [forwarded-for](#x-forwarded-for) | ["true", "false"] | "true" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [ingress.class](#ingress-class) | string | "" |  |:white_circle:|:large_blue_circle:|:white_circle:|
| [load-balance](#balance-algorithm) | string | "roundrobin" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [maxconn](#maximum-concurent-connections) | number |  |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [nbthread](#number-of-threads) | number | |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [pod-maxconn](#maximum-concurent-backend-connections) | number |  |  |:white_circle:|:white_circle:|:large_blue_circle:|
| [proxy-protocol](#proxy-protocol) | [IPs or CIDRs](#proxy-protocol) |   |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [rate-limit](#rate-limit) | "true"/"false" | "false" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [rate-limit-expire](#rate-limit) | string | "30m" | [rate-limit](#rate-limit) |:large_blue_circle:|:white_circle:|:white_circle:|
| [rate-limit-interval](#rate-limit) | string | "10s" | [rate-limit](#rate-limit) |:large_blue_circle:|:white_circle:|:white_circle:|
| [rate-limit-size](#rate-limit) | string | "100k" | [rate-limit](#rate-limit) |:large_blue_circle:|:white_circle:|:white_circle:|
| [request-capture](#request-capture) | string |  |  |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [request-capture-len](#request-capture) | string | "128" |  |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [request-set-header](#request-set-header) | string |  |  |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [server-ssl](#server-ssl) | ["true", "false"] | "false" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [servers-increment](#servers-slots-increment) | number | "42" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [ssl-certificate](#tls-secret) | string |  |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [ssl-passthrough](#https) | ["true", "false"] | "false" |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [ssl-redirect](#https) | "true"/"false" | "false" | [tls-secret](#tls-secret) |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [ssl-redirect-code](#https) | [301, 302, 303] | "302" | [tls-secret](#tls-secret) |:large_blue_circle:|:large_blue_circle:|:white_circle:|
| [syslog-server](#logging) | [syslog](#syslog-fields) | "address:127.0.0.1, facility: local0, level: notice" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-check](#timeouts) | [time](#time) |  |  |:large_blue_circle:|:large_blue_circle:|:large_blue_circle:|
| [timeout-client](#timeouts) | [time](#time) | "50s" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-connect](#timeouts) | [time](#time) | "5s" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-http-request](#timeouts) | [time](#time) | "5s" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-http-keep-alive](#timeouts) | [time](#time) | "1m" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-queue](#timeouts) | [time](#time) | "5s" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-server](#timeouts) | [time](#time) | "50s" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [timeout-tunnel](#timeouts) | [time](#time) | "1h" |  |:large_blue_circle:|:white_circle:|:white_circle:|
| [whitelist](#whitelist) | [IPs or CIDRs](#whitelist) | "" |  |:large_blue_circle:|:large_blue_circle:|:white_circle:|

#### Object specific annotations
| Annotation | Type | Default | Dependencies | Object |
| - |:-:|:-:|:-:|:-:|
| [weight](#traffic distribution) | number 0-256 | set via `--default-weight` parameter or 128 || Pod |

> :information_source: Annotations have hierarchy: `default` <- `Configmap` <- `Ingress` <- `Service`
>
> Service annotations have highest priority. If they are not defined, controller goes one level up until it finds value.
>
> This is usefull if we want, for instance, to change default behaviour, but want to keep default for some service. etc.

### Options

#### Access control

- Annotation: `blacklist`
  - Block given IPs and/or CIDR
- Annotation: `whitelist`
  - Allow only given IPs and/or CIDR
- Access control is disabled by default
- Access control can be set for all traffic (annotation on configmap) or for a set of hosts (annotation on ingress)
- `IPs or CIDR` - coma or space separated list of IP addresses or CIDRs

#### Balance Algorithm

- Annotation: `load-balance`
- use in format  `haproxy.org/load-balance: <algorithm> [ <arguments> ]`

#### Backend Checks

- Annotation: `check` - activate pod check (tcp checks by default)
- Annotation: [`check-http`](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-option%20httpchk) - Enable HTTP protocol to check on the pods health [`check` must be "true"]
  - uri: `check-http: "/check"`
  - method uri: `check-http: "HEAD /"`
  - method uri version: `check-http: "HEAD / HTTP/1.1\r\nHost:\ www"`
- Annotation: `check-interval` - interval between checks [`check` must be "true"]

#### Cookie persistence

- Configure sticky session via  cookie-based persistence.
- Annotation: `cookie-persistence <string>` sets the name of the cookie to be used for sticky session.
- More annotations to fine-tune cookie can be found in controller-annotations.go

More information can be found in the official HAProxy [documentation](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-cookie)

#### Request Capture

- Annotation: `request-capture`
  - Usage:
  ```
  request-capture: |
    <"request sample string">
    <"request sample string">
    ...
    ```
  where `<"request-sample-string">` looks like `"req.hdr(Host),lower,field(1,:)"`

- Annotation: `request-capture-len`
  - If this annotation is missing, default is `128`.
  - Usage:
  ```
  request-capture-len: <positive integer>
  ```

#### Request Set Header
- Annotation `request-set-header`
  - Single value:
    - Usage:
    ```
    request-set-header: <Header> <value>
    ```
    - Example:
    ```
    request-set-header: Host example.com
    ```
  - Multiple values:
    - Usage:
    ```
    request-set-header: |
      <Header> <value>
      <Header> <value>
    ```
    - Example:
    ```
    request-set-header: |
      Strict-Transport-Security "max-age=31536000"
      Cache-Control "no-store,no-cache,private"
    ```
#### Ingress Class

- Annotation: `ingress.class`
  - default: ""
  - used to monitor specific ingress objects in multiple controllers environment
  - any ingress object which have class specified and its different from one defined in [image arguments](controller.md) will be ignored

#### Https

- HAProxy will decrypt/offload HTTPS traffic if certificates are defined.
- Certificate can be defined in Ingress object: `spec.tls[].secretName`. Please see [tls-secret](#tls-secret) for format
- Annotation `ssl-passthrough`
  - by default ssl-passthrough is disabled.
	- Make HAProxy send TLS traffic directly to the backend instead of offloading it.
	- Traffic is proxied in TCP mode which makes unavailable a number of the controller annotations (requiring HTTP mode).
- Annotation `ssl-redirect`
  - redirects http trafic to https
  - by default, for an ingress with TLS enabled,  the controller redirects (302) to HTTPS.
	- Automatic redirects, when TLS enabled, can be disabled by setting annotation to "false" in configmap.
- Annotation `ssl-redirect-code`
  - HTTP status code on redirect
	- default is `302`

#### Maximum Concurent Frontend Connections

- Annotation: `maxconn`

#### Maximum Concurent Backend Connections

- Annotation: `pod-maxconn`
- related to backend servers (pods)

#### Number of threads

- Annotation: `nbthread`
- default value is number of procesors available

#### Proxy Protocol
- Annotation: `proxy-protocol`
- Enables Proxy Protocol for a list of IPs and/or CIDRs
- Connection will fait with `400 Bad Request` if source IP is in annotation list but no Proxy Protocol data is sent.
- usage:
  ```
	proxy-protocol: 192.168.1.0/24, 192.168.2.100
	```

#### Rate limit

Keep in mind this setting is global and will applied to all your traffic.
The number of requests a client can do per `rate-limit-interval` is **10**.

- Annotation: `rate-limit`
  - `true` / `false` - enable or disable rate limiting
- Annotation: `rate-limit-expire`
  - Table entries expire after `rate-limit-expire` of inactivity.
- Annotation: `rate-limit-interval`
  - request rate for the last `rate-limit-interval`
- Annotation: `rate-limit-size`
  - number of ip entries in table

#### Server ssl

- Annotation `server-ssl`
  - Use ssl for backend servers.
  - Current implementation does not verify server certificates.
- Example:
    `server server1 127.0.0.1:443 ssl verify none`

#### Servers slots increment

- Annotation `servers-increment`- determines how much backend servers should we
        put in `maintenance` mode so controller can
        dynamically insert new pods without hitless reload

#### Logging

- Annotation `syslog-server`: Takes one or more syslog entries separated by "newlines".
- Each syslog entry is a "comma" separated [syslog fields](#syslog-fields).
- "address" and "facility" are mandatory syslog fields.
- Example:
  - Single syslog server:

		syslog-server: address:127.0.0.1, port:514, facility:local0

  - Multiple syslog servers:

		syslog-server: |
			address:127.0.0.1, port:514, facility:local0
			address:192.168.1.1, port:514, facility:local1

- Logs can also be sent to stdout and viewed with `kubectl logs`:

		syslog-server: address:stdout, format: raw, facility:daemon

##### Syslog fields

The following syslog fields can be used:
- *address*:  Mandatory IP address where the syslog server is listening.
- *port*:     Optional port number where the syslog server is listening (default 514).
- *length*:   Optional maximum syslog line length.
- *format*:   Optional syslog format.
- *facility*: Mandatory, this can be one of the 24 syslog facilities.
- *level*:    Optional level to filter outgoing messages.
- *minlevel*: Optional minimum level.

More information can be found in the official HAProxy [documentation](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#3.1-log)

#### Timeouts

- Annotation `timeout-http-request`
- Annotation [`timeout-check`](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#timeout%20check)
- Annotation `timeout-connect`
- Annotation `timeout-client`
- Annotation `timeout-queue`
- Annotation `timeout-server`
- Annotation `timeout-tunnel`
- Annotation `timeout-http-keep-alive`

#### X-Forwarded-For

- Annotation: `forwarded-for`
- by default enabled, can be disabled per service or globally

#### Traffic distribution
- Annotation: `weight`
  - Set on the pod, this annotation can be used to set the weight of the pod in the backend configuration in order to control the amount of traffic these pods receive relative to other pods.
  - The default weight for all other endpoints or pods that don't have to specific annotation can be set via the controller command line parameter `--default-weight` and defaults to 128 if not set. 
  - More information can be found in the official HAProxy [documentation](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#5-weight) 

### Secrets

#### tls-secret

- define through pod arguments
  - `--default-ssl-certificate`=\<namespace\>/\<secret\>
- Annotation `ssl-certificate` in config map
  - \<namespace\>/\<secret\>
  - this replaces default certificate
- certificate can be defined in Ingress object: `spec.tls[].secretName`
- single certificate secret can contain two items:
  - tls.key
  - tls.crt
- certificate secret with `rsa` and `ecdsa` certificates:
  - :information_source: only one certificate is also acceptable setup
  - rsa.key
  - rsa.crt
  - ecdsa.key
  - ecdsa.crt

### Data types

#### Port

- value between <0, 65535]

#### Time

- number + type
- in miliseconds, "s" suffix denotes seconds
- example: "1s"
