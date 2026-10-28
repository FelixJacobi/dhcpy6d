# Authenticated Reconfigure Implementation Plan

Goal: Prompt opted-in DHCPv6 clients to renew after server restart or dynamic-prefix configuration, using RFC 8415 RKAP.

Approved design: authenticated Reconfigure with persistent per-client keys. Feature branch is based on upstream/config-prefix-substitution. NMU integration is deferred by user instruction.

Architecture: opt-in reconfigure_state_file stores per-server/client/interface keys, replay counters, endpoint and lease expiry in a private SQLite database independent of the lease backend. A worker sends individually authenticated Renew triggers using the existing UDP socket and bounded exponential retries; successful client renewals cancel retries. Key delivery is restricted to successful initial exchanges with Reconfigure Accept. No authentication bypass is enabled.

- [x] Add failing packet/authentication, persistence, expiry, retry and opt-in tests.
- [x] Implement durable RKAP state, packet generation, and retry worker in dhcpy6d/reconfigure.py.
- [x] Integrate key negotiation with final successful Replies, startup and prefix control commands; keep dry-run behavior.
- [x] Advertise Reconfigure Accept in stsbl-iserv-config-ipv6's generated dhcpcd config; add regression coverage.
- [x] Document configuration and first-renewal requirement; run Python and Perl suites and review changes.

Validation: independently decode DHCPv6 options and verify HMAC-MD5; reopen state to prove increasing replay values and stable keys; exercise unsuccessful replies, unknown/non-opted-in clients, expired state, changed server identity, retries and acknowledgements. Verify dhcpcd 9.4.1 authentication compatibility against source and, where available, live client behavior.

Validation: 31 Python tests passed, including the optional real dhcpcd auth.c checker; all 131 Perl assertions passed in config-ipv6. Bytecode compilation and git diff --check passed. The original checkout remains untouched.

Scope ruling: existing upstream Renew lease-selection and missing route-up hooks remain separate from the Reconfigure transport/authentication feature; documented in doc/reconfigure.rst. Mixed failed PD plus live NA does not acknowledge a pending Reconfigure.
