
# Using DuckDNS with Let's Encrypt

This guide explains how to issue wildcard TLS certificates for your DuckDNS domain using **acme.sh** and Let's Encrypt.

---

## Steps

1. **Create a DuckDNS account**
   - Visit [https://www.duckdns.org/](https://www.duckdns.org/).
   - Create an account and set up a subdomain.
   - Copy your **DuckDNS Token**.

2. **Install acme.sh and configure Let's Encrypt**

   ```bash
   curl https://get.acme.sh | sh
   source ~/.bashrc
   acme.sh --set-default-ca --server letsencrypt
   ```
   
3. **Generate certificates**

   ```bash
   acme.sh --issue --days 365 \
   --dns dns_duckdns \
   -d "*.openshifty.duckdns.org" \
   --cert-file ./openshifty.duckdns.org.crt \
   --key-file ./openshifty.duckdns.org.key \
   --ca-file ./ca.crt \
   --fullchain-file ./fullchain.crt \
   --debug
   
   acme.sh --issue --days 365 \
   --dns dns_duckdns \
   -d "*.apps.openshifty.duckdns.org" \
   --cert-file ./apps.openshifty.duckdns.org.crt \
   --key-file ./apps.openshifty.duckdns.org.key \
   --ca-file ./ca.crt \
   --fullchain-file ./apps-fullchain.crt \
   --debug
   ```
