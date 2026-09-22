# Fork changes vs upstream

This fork (`matttheplanet/ax-fork`) diverges from upstream `attacksurge/ax`
(`upstream/master`) with the fixes below, grouped by theme. Regenerate the
authoritative list any time with:

```bash
git fetch upstream
git log --oneline upstream/master..master
```

## Image build — tool dependencies & OS

- **nmap can actually load** (`475d0f8`, `1b9348d`) — nmap is installed from the
  nmap.org RPM via `alien` + `dpkg -i`, which does **not** resolve dependencies.
  That build links `libblas.so.3`; on the 24.04 base it isn't present by default,
  so nmap failed at load (`error while loading shared libraries: libblas.so.3`)
  and returned silent empty scans. Fix: explicitly install `libblas3`/`liblapack3`,
  run `apt-get -f install`, and add a `nmap --version` build-time check (fails the
  build loudly if nmap still can't load) — applied to **all four** provisioners
  (default / extras / reconftw / barebones).
- **Ubuntu 24.04 base** (`16c132b`) — move IBM VPC to 24.04; make the default
  provisioner version-agnostic.
- **24.04 compatibility** (`d6177ef`) — allow the Chrome sandbox; make
  reconftw/extras/barebones pip installs 24.04-safe.
- **Go 1.25** (`82b214f`) — bump Go 1.23.0 → 1.25.0.
- **AWS disk size** (`9948f3e`) — default build disk 20 → 50 GB (builds ran out
  of space).

## IBM VPC support

- **`resource_group_id` capture + wiring** (`0c8a475`) — setup collected the
  resource group by NAME only; the `ibmcloud-vpc` builder never received a
  resource group, so builds fell back to the default RG. Setup now resolves the
  RG's ID (`ibmcloud resource group <name> --id`) and persists `resource_group_id`
  in the account JSON; the builder (json + pkr.hcl) passes it to the source.
- **Setup UX + validation** (`5ed5170`) — clearer prompts, validate region/RG
  names, guard empty `subnet_id`.
- **SSH-readiness gating** (`835bcc7`) — gate instance readiness on SSH
  reachability.
- **Build as `ubuntu`, not `root`** (`deb9c1d`).
- **Builder timeout** (`ef3aea0`) — 50m → 90m for the larger 24.04 image.
- Provider runtime changes in `providers/ibm-vpc-functions.sh`.

## Provisioner correctness / robustness

- **`set -e`** (`f412d78`) — restore so failed installs fail the build.
- **Docker steps guarded** (`832cebc`) — `|| true` on docker image-build steps.
- **trufflehog / webscreenshot** (`91655cf`) — trufflehog main branch,
  webscreenshot system install.
- **extras clone URL** (`672a329`) — clone `ax.git`, not the 404 `axiom.git`.
- **ssh service name** (`c0fa2d2`) — restart `ssh`, not `sshd` (Ubuntu unit is
  `ssh.service`).
- **file ownership** (`67b9b06`, `084de28`) — root-own the moved `sshd_config` /
  `00-header`; chown sudoers as root, not pkexec (broke Ubuntu login).

## Fleet reliability (scan / exec runtime)

- **Preflight retry** (`823e8d5`) — retry unreachable instances instead of
  pruning them.
- **SSH-readiness** gating in `axiom-scan` / `axiom-exec`.
- **`clean_up` exit code** (`b654f17`) — axiom-scan no longer leaks the
  terminal-reset exit code from `clean_up`.

## Fork housekeeping

- **Self-clone URLs** (`733649f`) — point ax's self-clones at
  `matttheplanet/ax-fork` for testing.
- **README** (`1eebc5b`).
