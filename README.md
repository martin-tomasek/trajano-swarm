# Trajano base Docker swarm stacks

This contains the stack files used to deploy my Docker swarm.  It uses Traefik 2.2 to do the TLS routing and SSL termination and client certificate validation and another Traefik to manage the intranet services.  

It uses the label `intranet=true` to distinguish intranet services from external services.

In addition, this has a management plane which provides Zipkin and Portainer agents and a management UI stack exposed to the intranet to access the necessary info.

Due to Traefik only not allowing TLS options to be set in Docker labels, a TOML file containing the configuration is needed in order to at least get to **SSLLabs A rating**.  This configuration file `traefik.toml` contains common middlewares that apply to both public and intranet traefiks.  The `/ping` endpoint is where I stored the labels used to dynamically configure the Docker provider that are specific to the environment.  

## Common middlewares

* `compress-all` which provides compression.
* `security-headers` which provides additional headers to [push Trajano.net SSLLabs to A+ rating](https://www.ssllabs.com/ssltest/analyze.html?d=trajano.net)
* `strip-prefix` which strips the prefix and does a redirect if the first segment does not end with `/` which is useful for reverse proxies that have a single DNS with top level path per application.

The `default` middleware chain for public is `https-only@file,security-headers@file,compress-all@file`

The `default` middleware chain for intranet is `compress-all@file` since it is expected the internal services are HTTP only from the point of view of the Apache proxy.

## Required files

The mappings to the following file should change to match your Intranet CA and the traefik configuration file respectively.

    - /d/p/trajano.net/devops-ca-R2.crt
    - /d/p/trajano.net/traefik.toml

## Deploying

```
docker stack deploy -c management.yml --prune management
docker stack deploy -c edge.yml --prune edge
docker stack deploy -c intranet.yml --prune intranet
docker stack deploy -c management-ui.yml --prune management-ui
```

## Notes

* External dashboard is exposed to `44444` for now until https://github.com/containous/traefik/issues/5853 which will allow a second dashboard and API endpoint on the same domain.
* Intranet dashboard is exposed to "/traefik" based on comment by https://github.com/containous/traefik/issues/5853#issuecomment-575290336.
* `setup-elk.sh` configures the ELK stack after it has been deployed.

## DMZs

In the most recent refactoring, having an intermediate DMZ server was removed and now it is Traefik to Traefik communication for the intranet.  However, this approach prevents having complex rules or transformations that would've been possible with either Apache or nginx.

## Branches
* [apache](https://github.com/trajano/trajano-swarm/tree/apache) uses Apache HTTPd as the DMZ.
* [nginx](https://github.com/trajano/trajano-swarm/tree/nginx) uses nginx as the DMZ.
