
[![Container Repository on Quay](https://img.shields.io/badge/quay.io-eas-green "Container Repository on Quay")](https://quay.io/repository/wi_stefan/endpoint-configuration-service?tab=tags)
[![Coverage Status](https://coveralls.io/repos/github/wistefan/endpoint-auth-service/badge.svg?branch=initial-dev)](https://coveralls.io/github/wistefan/endpoint-auth-service?branch=initial-dev)
[![Unit-Test](https://github.com/wistefan/endpoint-auth-service/actions/workflows/unit.yml/badge.svg)](https://github.com/wistefan/endpoint-auth-service/actions/workflows/unit.yml)
[![Integration-test](https://github.com/wistefan/endpoint-auth-service/actions/workflows/it.yml/badge.svg)](https://github.com/wistefan/endpoint-auth-service/actions/workflows/it.yml)
-------

# Endpoint-Auth-Service

In various use-cases, there is a need to apply authn/z to outgoing requests for components that do not handle this them-self(f.e. notifications in
[NGSI-LD brokers](https://github.com/FIWARE/context.Orion-LD)). This sidecar provides that by adding an [envoy-proxy](https://www.envoyproxy.io) 
as [sidecar](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/ch02.html) to the component that gets forwarded all 
outgoing requests via ip-tables(see [iptables-init](./src/iptables-init)). The sidecar-proxy does request auth-information at the [auth-provider](./src/auth-provider) 
and adds it to the requests accordingly. The endpoints to be handled and there auth-information can be configured through
[endpoint-configuration-service](./src/endpoint-configuration-service).

## Overview

![Proxy-Architecture](./doc/img/arch-overview.svg)

The architecture consists of 2 main components:
- the sidecar-proxy to intercept and manipulate outgoing requests
- the endpoint-auth-service to provide configuration and authentication information

This architecture allows to separate the actual authentication flows from the proxy itself, thus giving more flexibility in terms of technology
and reduces the complexity of the lua-code inside the proxy. All lua-code placed there needs to be non-blocking to not kill the performance of the 
proxy(and therefore the proxied requests itself). Most auth-flows require contacting external services(f.e. the IDP in the iShare use-case) or some 
file-io(f.e. reading the certs in iShare), which is hard to implement in a non-blocking fashion. With using the built-in http-call method and moving 
the complexity into the domain of the auth-provider, this can be avoided. Besided that, it allows to use request-caching for thos calls to the auth-provider
in order to prevent a new auth-flow for every request.

## ADRs

- [Add authentication via a sidecar-proxy](./doc/adr/sidecar-based-auth.md)
- [Use envoy as proxy](./doc/adr/choose-proxy.md)
- [Use mustache templating for envoy config](./doc/adr/mustache-templating.md)
- [Implement auth-providers as separate components](./doc/adr/authprovider-as-separate-component.md)
- [Auth-provider config on path level](./doc/adr/auth-provider-on-path-level.md)

## APIs

- [Endpoint-Configuration-API](./api/endpoint-configuration-api.yaml)
- [Auth-Provider-API](./api/auth-provider-api.yaml)
- [iShare-Credentials-Management-API](./api/ishare-credentials-management-api.yaml)

## Why not use mTLS?

TODO: describe that mTLS is just another option and can be used together, outline similarities to improve understandability