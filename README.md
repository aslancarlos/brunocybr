# brunocybr

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Bruno](https://img.shields.io/badge/Bruno-collection-f4a72b)](https://www.usebruno.com/)
[![CyberArk Secrets Manager SaaS](https://img.shields.io/badge/CyberArk-Secrets%20Manager%20SaaS-0B7285)](https://docs.cyberark.com/secrets-manager-saas/)

Collection [Bruno](https://www.usebruno.com/) **completa** para a **CyberArk Secrets
Manager - SaaS** (Conjur Cloud) — autenticação, secrets, policies, resources, roles,
host factory, certificate authority e as APIs **v2** de workloads/safes/autenticadores.

Bruno é um cliente HTTP open-source, baseado em arquivos (alternativa ao Postman), que
guarda a collection em texto plano (`.bru`) — versionável e diffável no git.

## Conteúdo (46 requests em 10 pastas)

| Pasta | Requests |
|---|---|
| **Health and Status** | health, info, whoami, list authenticators, authenticator status (service/GCP), remote health |
| **Authentication** | login (API key), authenticate (access token), rotate API key, change password, authn-jwt / authn-iam / authn-azure / authn-gcp / authn-oidc (CyberArk Identity) / authn-k8s / authn-ldap, enable/disable authenticator |
| **Secrets** | retrieve secret, set secret value, batch retrieve |
| **Policies** | append (POST), replace (PUT), update (PATCH) |
| **Resources** | list (global / por account / por kind), show resource |
| **Roles** | show role, add/remove membership |
| **Host Factory** | create token, revoke token, create host |
| **Public Keys** | show public keys |
| **Certificate Authority** | sign certificate (CSR) |
| **SaaS v2 APIs** | batch retrieve (até 250), add group member, update authenticator, create/delete workload |

Base v1 gerada a partir do [OpenAPI oficial do Conjur](https://github.com/cyberark/conjur-openapi-spec);
adições v2 conforme a [documentação Secrets Manager SaaS](https://docs.cyberark.com/secrets-manager-saas/latest/en/content/developer/lp_rest_api.htm).

## Como usar

1. Instale o [Bruno](https://www.usebruno.com/downloads).
2. Clone este repositório.
3. No Bruno: **Open Collection** → aponte para a pasta clonada.
4. Selecione o environment **SaaS Example** (canto superior direito) e ajuste os
   valores para o seu tenant — veja a tabela abaixo.
5. Rode na ordem: **LOGIN (get API key)** → **AUTHENTICATE (get access token)**.
   Os scripts guardam `apiKey` e `accessToken` no environment automaticamente; os demais
   requests já usam o header `Authorization: Token token="{{accessToken}}"`.

> Para workloads (não-humanos), pule o LOGIN e rode direto o `authn-jwt` / `authn-iam` /
> `authn-oidc` correspondente — o access token é gravado da mesma forma.

## Environment (`environments/SaaS Example.bru`) — variáveis e exemplos

Todos os valores são **exemplos genéricos**; troque pelos do seu ambiente.

| Variável | Exemplo | Observação |
|---|---|---|
| `baseUrl` | `https://example-tenant.secretsmgr.cyberark.cloud/api` | URL da API do tenant |
| `account` | `conjur` | account do Conjur Cloud (normalmente `conjur`) |
| `login` | `admin` | usuário ou host id |
| `password` | `ChangeMe-Example-Passw0rd` | senha (Basic Auth do LOGIN) |
| `apiKey` | *(preenchido pelo LOGIN)* | |
| `accessToken` | *(preenchido pelo AUTHENTICATE)* | usado no header `Token token="..."` |
| `variableKind` / `variableId` | `variable` / `data%2Fvault%2Fmy-safe%2Fdbuser_dual%2Fpassword` | id do secret **URL-encoded** (`/` → `%2F`) |
| `policyId` | `data/vault/my-safe` | branch da policy (barras literais) |
| `serviceId` | `my-jwt-authenticator` | service_id do autenticador |
| `oidcServiceId` | `cyberark` | service_id OIDC do CyberArk Identity |
| `workloadId` | `host%2Fdata%2Fvault%2Fmy-safe%2Fmy-app` | host id URL-encoded |
| `jwt` / `identityToken` / `azureAccessToken` / `gcpIdentityToken` | *(tokens de exemplo)* | credenciais de workload por nuvem |
| `authnType` / `authnName` | `authn-iam` / `default` | usados nas APIs v2 |
| `workloadName` / `safeName` | `my-app` / `my-safe` | usados nas APIs v2 |
| `roleKind` / `roleId` / `memberId` | `user` / `alice` / `my-safe:host:my-app` | roles e membership |
| `hostFactoryToken` / `remote` | *(exemplos)* | host factory / follower |

## Notas importantes

- **Access token**: os requests de `authenticate` usam `Accept-Encoding: base64`, então o
  corpo da resposta já é o token pronto para o header `Authorization: Token token="..."`.
- **URL-encoding**: ids de secret (`/secrets/...`) precisam ser URL-encoded; ids de policy
  (`/policies/...`) usam barras literais.
- **APIs v2**: exigem o header `Accept: application/x.secretsmgr.v2+json`. Alguns contratos
  v2 (create workload, batch v2) podem variar por versão do tenant — confira a doc v2 e a
  nota em `docs` de cada request.
- **`conjur_enterprise.json`**: export legado (Conjur Enterprise) mantido para referência;
  a collection nativa `.bru` deste repo é a versão SaaS completa e recomendada.

## Segurança

Este repositório contém apenas **valores de exemplo** — nenhuma credencial, endpoint ou
token real. Não faça commit de segredos reais; use o environment local do Bruno para isso.

## Licença

Apache License 2.0 — veja [LICENSE](LICENSE).
