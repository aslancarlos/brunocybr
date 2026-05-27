# brunocybr

A [Bruno](https://www.usebruno.com/) HTTP client collection covering common **CyberArk** APIs — Conjur Enterprise authentication, secret retrieval, and policy management. Use it as a starting point when you need to script against the Conjur REST API or quickly try a call from a workstation without writing code.

Bruno is an open-source, file-based alternative to Postman that stores collections as plain text, so the contents of this repo can be diffed and version-controlled normally.

## What's in here

- **Auth** — login, exchange API key for short-lived access token
- **Secrets** — fetch single and batch secrets
- **Policies** — load and inspect policy
- *(extend as needed for your environment)*

## Using the collection

1. Install [Bruno](https://www.usebruno.com/downloads).
2. Clone this repository.
3. In Bruno, choose **Open Collection** and point it at the cloned folder.
4. Edit the environment variables (`CONJUR_URL`, `ACCOUNT`, `ADMIN`, `PASSWORD`) to match your tenant.
5. Run the **LOGIN** request first to populate the access token.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
