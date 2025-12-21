# Code Review

## Security
- **LDAP filter injection risk.** User-controlled usernames are interpolated directly into LDAP search filters without escaping, e.g., `(uid=$in_username)` in both `Authenticate()` and `DoSearchUsername()`. An attacker could craft a username containing LDAP filter operators to bypass group checks or retrieve unintended records. Use `ldap_escape()` (WordPress 6.2+ bundles it) or a library helper before building filters, and reject malformed values up front. 【F:lib/ldap_ro.php†L41-L75】【F:lib/ldap_ro.php†L143-L169】
- **SSO trusts request headers for auto‑provisioning.** When SSO is enabled, `wpmuLdapSSOAuthenticate()` pulls a username from `LOGON_USER`/`REMOTE_USER`/`AUTH_USER` headers and will create or log in that account (and optionally create a site) without verifying the source of the header. Unless a trusted reverse proxy strips/sets these headers, this is vulnerable to spoofing. Require server‑side verification (e.g., authenticated reverse proxy), restrict auto‑provisioning to allowed groups, and validate the header format before use. 【F:lib/wpmu_ldap.functions.php†L358-L381】【F:lib/wpmu_ldap.functions.php†L149-L211】

## Reliability
- **Undefined password helper.** `generate_random_password()` is called when creating users but is never defined or required in the plugin, which will trigger a fatal error during account creation. Replace with WordPress’ `wp_generate_password()` (or include a helper) to ensure new accounts are provisioned successfully. 【F:lib/wpmu_ldap.functions.php†L33-L92】
- **Deprecated user meta APIs.** The plugin relies on `update_usermeta()`/`get_usermeta()`, which have been deprecated for years in favor of `update_user_meta()`/`get_user_meta()`. Continuing to use the deprecated APIs risks compatibility issues with newer WordPress versions and makes deprecation warnings harder to triage. Refactor to the modern meta functions. 【F:lib/wpmu_ldap.functions.php†L41-L327】

## Maintainability
- **Use of `extract()` on untrusted arrays.** Several entry points call `extract($opts)` on caller‑provided arrays, introducing the risk of variable shadowing and making the function parameters implicit. Converting these to explicit local assignments (or `wp_parse_args()` with a defaults array) will improve readability and reduce accidental overrides. 【F:lib/wpmu_ldap.functions.php†L12-L277】
