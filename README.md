# brunocybr

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Bruno](https://img.shields.io/badge/Bruno-collection-f4a72b)](https://www.usebruno.com/)
[![CyberArk Secrets Manager SaaS](https://img.shields.io/badge/CyberArk-Secrets%20Manager%20SaaS-0B7285)](https://docs.cyberark.com/secrets-manager-saas/)

Collection [Bruno](https://www.usebruno.com/) **completa** para a **CyberArk Secrets
Manager - SaaS** (Conjur Cloud) — autenticação, secrets, policies, resources, roles,
host factory, certificate authority e as APIs **v2** de workloads/safes/autenticadores.

Bruno é um cliente HTTP open-source, baseado em arquivos (alternativa ao Postman), que
guarda a collection em texto plano (`.bru`) — versionável e diffável no git.

## Conteúdo (56 requests em 11 pastas)

| Pasta | Requests |
|---|---|
| **Setup** | **Tenant Discovery (auto-fill)** — preencha só o `subdomain` e ele descobre e preenche `baseUrl`, `pcloudUrl`, `identityUrl`, `account`, `tenantId`, `identityId` |
| **Health and Status** | health, info, whoami, list authenticators, authenticator status (service/GCP), remote health |
| **Authentication** | login (API key), authenticate (access token), rotate API key, change password, authn-jwt / **authn-iam (AWS)** / authn-azure / authn-gcp / authn-oidc (CyberArk Identity) / authn-k8s / authn-ldap, enable/disable authenticator |
| &nbsp;&nbsp;↳ **PCloud (ISPSS)** *(subpasta de Authentication)* | get platform token (service user / IAM), token via OAuth2 app alias, List Safes (uso do bearer), exchange platform token → Conjur (authn-oidc) |
| &nbsp;&nbsp;↳ **CyberArk Identity Login (MFA)** *(subpasta de Authentication)* | login do **usuário padrão com MFA**: 1. StartAuthentication → 2. Advance (senha) → 3a. start MFA (SMS/e-mail) / 3b. answer MFA (OTP/TOTP) / 3c. poll (push) |
| **Secrets** | retrieve secret, set secret value, batch retrieve |
| **Policies** | append (POST), replace (PUT), update (PATCH) |
| **Resources** | list (global / por account / por kind), show resource |
| **Roles** | show role, add/remove membership |
| **Host Factory** | create token, revoke token, create host |
| **Public Keys** | show public keys |
| **Certificate Authority** | sign certificate (CSR) |
| **SaaS v2 APIs** | batch retrieve (até 250), add group member, update authenticator, create/delete workload |

### Formas de autenticar

- **Usuário padrão com MFA** (interativo — recomendado para humanos):
  `Authentication → CyberArk Identity Login (MFA)`. Fluxo `StartAuthentication` →
  `AdvanceAuthentication` (senha → MFA: OTP/TOTP/SMS/e-mail/push). Não precisa de service
  user OAuth. Gera `identityBearer`. **Nota:** se o `StartAuthentication` responder com um
  `PodFqdn`, ajuste `identityUrl` para `https://<PodFqdn>` e repita (roteamento de pod do
  CyberArk Identity). Tenants antigos usam o domínio `.my.idaptive.app`.
- **AWS IAM** (workload): `Authentication → authn-iam` — troca os headers assinados de um
  `STS GetCallerIdentity` por um access token do Conjur.
- **Service user / IAM do PCloud** (CyberArk Identity / ISPSS): `Authentication → PCloud
  (ISPSS) → Get platform token` — OAuth2 `client_credentials` no `/oauth2/platformtoken`,
  bearer para as APIs do Privilege Cloud (fluxo do `cybr-cli -a identity` / psPAS).
  ⚠️ Exige um **service user** dedicado, não funciona com usuário interativo/humano.

Base v1 gerada a partir do [OpenAPI oficial do Conjur](https://github.com/cyberark/conjur-openapi-spec);
adições v2 conforme a [documentação Secrets Manager SaaS](https://docs.cyberark.com/secrets-manager-saas/latest/en/content/developer/lp_rest_api.htm).

## Como usar

1. Instale o [Bruno](https://www.usebruno.com/downloads).
2. Clone este repositório.
3. No Bruno: **Open Collection** → aponte para a pasta clonada.
4. Selecione o environment **SaaS Example** e preencha **apenas o `subdomain`**
   (ex.: `latamlab`). Rode **`Setup → Tenant Discovery (auto-fill)`** — ele preenche
   sozinho `baseUrl`, `pcloudUrl`, `identityUrl`, `account`, `oidcServiceId`, `tenantId`
   e `identityId` a partir do endpoint público de discovery da CyberArk.
5. Autentique conforme o caso:
   - **Você (humano, com MFA):** `Authentication → CyberArk Identity Login (MFA)` →
     `1. Start authentication` → `2. Advance (senha)` → `3b. Advance (OTP/TOTP)`
     (ou `3a`/`3c` para SMS-email/push).
   - **Workload/host:** rode `authn-jwt` / `authn-iam` / etc.
   - **Conjur nativo:** `LOGIN` → `AUTHENTICATE` (não vale para usuário SaaS — use MFA/OIDC).
   Os scripts guardam os tokens (`accessToken`, `identityBearer`, …) no environment.

> Para workloads (não-humanos), pule o LOGIN e rode direto o `authn-jwt` / `authn-iam` /
> `authn-oidc` correspondente — o access token é gravado da mesma forma.

## Environment (`environments/SaaS Example.bru`) — variáveis e exemplos

Na prática **você só precisa preencher `subdomain`** e rodar o Tenant Discovery — o resto
é preenchido sozinho. As demais variáveis abaixo são só para requests específicos.

| Variável | Exemplo | Observação |
|---|---|---|
| **`subdomain`** ⭐ | `latamlab` | **a única obrigatória** — o Tenant Discovery deriva o resto |
| `baseUrl` | `https://example-tenant.secretsmgr.cyberark.cloud/api` | 🔄 auto (Discovery) — URL da API do Conjur |
| `pcloudUrl` / `identityUrl` | — | 🔄 auto (Discovery) |
| `tenantId` / `identityId` | — | 🔄 auto (Discovery) |
| `account` | `conjur` | 🔄 auto (Discovery) — normalmente `conjur` |
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
| `identityUrl` | `https://example-tenant.id.cyberark.cloud` | tenant do CyberArk Identity (ISPSS) |
| `pcloudUrl` | `https://example-tenant.privilegecloud.cyberark.cloud` | tenant do Privilege Cloud |
| `pcloudServiceUser` / `pcloudServiceSecret` | `svc-conjur-api@example-tenant` / *(exemplo)* | usuário IAM/serviço + segredo (client_credentials) |
| `pcloudAppAlias` | `__idaptive_cybr_user_oidc` | app alias do OAuth2 client (variante) |
| `pcloudToken` | *(preenchido pelo Get platform token)* | bearer para as APIs do PCloud |
| `idUser` | `user@example.com` | seu usuário padrão do CyberArk Identity (login MFA) |
| `idPassword` | *(sua senha)* | senha do usuário no fluxo MFA |
| `idOtp` | *(código do MFA)* | OTP/TOTP do SMS/e-mail/app autenticador |
| `idSessionId` / `idTenantId` / `idMechPassword` / `idMechMfa` | *(preenchidos pelo StartAuthentication)* | sessão + ids dos mecanismos |
| `identityBearer` | *(preenchido no LoginSuccess)* | token de sessão do CyberArk Identity |

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
