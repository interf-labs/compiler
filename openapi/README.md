# OpenAPI

`local-service.openapi.json` is the one published OpenAPI document, regenerated
from the Runtime's own route and schema owners. Treat it as the public route
list; this folder should not carry a second hand-maintained route list.

It is a promise to integrators, so it is narrower than the service. Operations
that only an Interf account holder can exercise are served by the local service
but withheld from this file. The maintainer generator carries the allowlist and
the reason.

Do not publish retired reusable-layer routes or compatibility aliases in the
generated OpenAPI output.
