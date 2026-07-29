---
title: oauth
excerpt: |-
  OAuth 2.1 authorization server for MCP custom connectors.
  RFC 8414 metadata + RFC 7591 anonymous dynamic client registration;
  authorize/token/revoke land in later rows. The whole surface 404s
  while OAUTH_AS_ENABLED is off. Network-agnostic: mode is carried
  inside the minted token, so X-Network does not apply here.
hidden: false
---