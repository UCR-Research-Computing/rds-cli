# rds-cli: specification

| | |
|---|---|
| Spec version | 1.1, 2026-11-04 (version metadata caught up to tag v0.1.14; 1.0 first spec same day) |
| Applies to | rds-cli v0.1.14 (pyproject, tag v0.1.14, CHANGELOG 2026-10-28; docs-only release over 0.1.13 code) |
| Status | Released; installed by researchers with `uv tool install git+https://github.com/UCR-Research-Computing/rds-cli.git` |
| Owner | Chuck Forsyth (Research Computing). Authors: Chuck and AI assistants. |
| Repo | https://github.com/UCR-Research-Computing/rds-cli (public; transferred from charles-forsyth, PR #14) |
| Related | CephRDS service (Nexus record "CephRDS"; RC site /services/cephrds/); ceph-rds-agent (repo charles-forsyth/cephrds-connect) |

## 1. What rds-cli is

A command-line client for CephRDS, UCR's S3-compatible research data storage (endpoint
`https://rds.ucr.edu`, reachable on campus or campus VPN only). It lets researchers list, upload,
download, inspect and move data, including whole directories in parallel, and stream single files
between CephRDS and Google Cloud Storage.

## 2. Layout

| Path | What |
|---|---|
| `src/rds_cli/main.py` | Typer app, entry point `rds-cli = "rds_cli.main:app"` |
| `src/rds_cli/client.py` | S3 operations (boto3, path-style), parallel transfers, GCS streaming |
| `src/rds_cli/config.py` | credentials: `rds-cli auth` writes `~/.config/rds-cli/.env` (mode 0600); falls back to AWS_* env vars |
| `src/rds_cli/utils.py` | formatting, path-safety helpers |
| `tests/` | test_client, test_config, test_main, test_robustness, test_utils |

## 3. Behaviour that must hold

1. Credentials: interactive `auth` saves to `~/.config/rds-cli/.env` with 0600; otherwise
   `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_ENDPOINT_URL`. Keys are never printed.
2. Directory transfers run in parallel (`concurrent.futures`); one failing file is reported and
   does not abort the rest.
3. Recursive downloads validate every resolved path stays inside the destination (traversal guard).
4. Listings are bounded (`--limit`, default 1000) so huge buckets cannot exhaust memory.
5. Cross-cloud copy streams one file at a time through a temp file between S3 and `gs://`.
6. Commands (README table): `auth`, `bucket-info` (alias `info`, deprecated), `file-info` (alias
   `stat`, deprecated), `ls`, `upload`, `download`, `cp`, `mv`, `rm`, `share` (presigned, expiring
   links; anyone holding the link can read the object until it expires, within the campus/VPN
   reach of rds.ucr.edu).
7. `ls --limit -1` lists a whole bucket; the default stays bounded.

## 4. Policy context

CephRDS is S3 only (no NFS), on campus network or campus VPN only, no lifecycle rules. Bucket
sharing with outside people goes through an RC request. Researchers keep their own backups.

## 5. Open items

1. ~~CHANGELOG had no entry after 0.1.13~~ Fixed 2026-11-04: 0.1.14 entry (CITATION.cff, org move; no code change) and pyproject now 0.1.14.
2. `share` hands out presigned links; confirm that fits the CephRDS sharing policy (sharing with outside people is by RC request).
3. Consider listing rds-cli on the CephRDS service page and Nexus record as the recommended client.

## Change log

- 2026-11-04 1.0: first spec.
