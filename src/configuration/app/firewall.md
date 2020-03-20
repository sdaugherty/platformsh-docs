# Outbound firewall

In some situations, compliance regulations may require you to limit outbound traffic from your application.  The `firewall` property allows you to do so.

This setting has no impact on inbound requests to your application.  For that, use the environment access control settings in the Management Console.

## Syntax

The `firewall` property defines one or more whitelist entries for outbound requests.  It's basic syntax is as follows:

```yaml
firewall:
  outbound:
    - protocol: tcp
      ips: ["1.1.1.1/32"]
      ports: [443]
    - protocol: tcp
      ips: ["1.2.3.4/32"]
      ports: [443]
```

The above example allows two outbound rules over TCP.  All other outbound requests will be blocked and will time out eventually (usually after 30 seconds).

If no rules are specified, the default `firewall` configuration is equivalent to:

```yaml
firewall:
  outbound:
    - protocol: tcp
      ips: ["0.0.0.0/0"]
```

That is, all outbound TCP traffic is allowed on all ports (aside from port 25, which is always blocked without exception).  In the majority of cases the default is sufficient for most applications.

## Options

Each firewall rule has three configuration values.

### `protocol`

The protocol value is always `tcp`.  Outbound UDP request are not allowed anyway.  As a result this property can be omitted in virtually every circumstance.

### `ips`

This property is an array of IP addresses in [CIDR notation](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).  CIDR allows you to specify a range of IP addresses in a compact format, using a bitmask.  Most commonly the bitmask is 8, 16, or 32 but that is not required.

For example, `1.2.3.4/8` will match any IP address whose first 8 bits match `1.2.3.4`, which corresponds to the first segment.  Therefore it will allow `1.*.*.*`.  In comparison, `1.2.3.4/24` will allow `1.2.3.*`.  A mask of 32 will match only the IP address specified, so to whitelist a single specific IP you must write `1.2.3.4/32`.

[IP Address Guide](https://ipaddressguide.com/cidr) has a useful CIDR format calculator.

This is the only required property.

### `port`

To restrict a rule to only allow requests to certain ports as well, list the ports in this property.  For example, `[80, 443]` will only allow requests to the specified IPs on ports 80 and 443 (typically HTTP and HTTPS, respectively).  Requests to any other port will be blocked.

If not specified, requests to a given IP may be to any port.  Legal values are integers from 1 to 65535.

## Usage considerations

Be aware that many services your application may wish to connect to will be using a domain name that is not on a fixed IP address, or is load-balanced between multiple IP addresses.  You will need to contact the administrator of that service in order to determine the correct IP addresses to whitelist.

Also be aware that many services are behind a Content Delivery Network (CDN).  For most CDNs, routing is done via domain name, not IP address, so thousands of domain names may share the same public IP addresses at the CDN.  If you whitelist the IP address of a CDN, you will in most cases be whitelisting many or all of the other customers hosted behind that CDN.  That has security implications and limits the usefulness of this configuration option.
