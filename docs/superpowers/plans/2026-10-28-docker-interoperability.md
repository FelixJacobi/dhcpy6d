# Docker DHCPv6 interoperability test plan

Goal: exercise actual dhcpy6d and Kea perfdhcp, ISC dhclient, dhcpcd and systemd-networkd on an isolated IPv6 link; run the same suite in GitHub Actions.

User-approved coverage: direct and relay-generated Kea traffic, address/prefix allocation, renewal/rebinding, release, restart, prefix change and authenticated Reconfigure where supported. Kea perfdhcp provides relay simulation (-A 1), not a standalone Kea relay daemon.

Architecture: a privileged Docker container with --network none owns two private network namespaces joined by a veth pair. Processes are real distro-packaged clients; server runs checkout code. Capture every DHCPv6 packet and independently assert exchanges and resource contents, persistent database state, client kernel state where applicable, and bounded recovery. Save packet traces, logs, versions and JUnit results. Explicit capability skips and narrowly documented expected failures remain visible.

- [x] Build Debian bookworm image with clients and capture tooling.
- [x] Implement namespace/process fixtures, packet decoding, independent assertions and per-case artifacts.
- [x] Implement client and feature matrix, including real authentication/replay checks and known upstream gaps.
- [x] Add local Docker wrapper and GitHub Actions matrix/artifact upload.
- [x] Run the complete Docker suite locally and unit tests; independently review and commit.

Constraints: no host networking or Docker socket inside container. No production state mounted. Original conflicted checkout and NMU remain untouched. Keep this work on the authenticated Reconfigure feature branch.

Validation: final Docker matrix exercised 52 cases: 31 passed, 16 explicit capability skips, 5 known upstream failures (four relay paths, one Renew route-hook gap). Live dhcpcd mixed-IA renumbering, authenticated trigger, tamper/replay rejection and counter persistence passed. All 32 Python unit tests passed including the optional real dhcpcd C authentication checker. All 132 config-ipv6 Perl assertions passed. Workflow YAML, shell syntax, Python compilation and diff whitespace checks passed. GitHub Actions has been added but has not run remotely.

Docker execution exposed and regression-tested the SQLite backend import-order initialization bug. It also exposed the dhcpcd IPv4-to-IPv6 option-mapping interaction; a separate config-ipv6 commit preserves explicit DNS/search/Rapid Commit requests. Review strengthened persistence, release, DNS content, kernel-route and liveness checks.
