# redteam_lean

An operator workstation for authorized Windows/Active Directory assessments,
with a small web testing toolkit. Based on the catalog in `mapdocs/map.md` and
checked against the module sources and `tools_config.json`.

## Install

Run from this checkout on Linux (preferably Kali/Debian/Ubuntu), or WSL2.
On macOS, use a Linux VM. This profile assesses Windows targets; it does not
install tools into native Windows or macOS.

```bash
# Review the exact selection without installing anything.
bash install.sh --profile redteam_lean --dry-run

# Install the selection and its prerequisites.
sudo bash install.sh --profile redteam_lean
```

The profile selects **29 catalog tools** across five modules. `MODULES` groups
the catalog for discovery; `TOOLS` controls installation. Whole modules are
not installed. Sliver's client and server are included, with `INCLUDE_C2=true`.
Heavy module bundles, Docker images, password analysis tools and other C2/phishing
frameworks are excluded. Adding `--enable-docker` or `--include-c2` does not expand
this explicit selection; use a separate installation for other additions.

The installer also installs Git, curl, build tools, OpenSSL development headers,
Ruby and Ruby headers. Go, Python/pipx and Rust toolchains are bootstrapped by
the relevant installers when missing. These dependencies add disk usage beyond
the 29 tools. RustHound CE is compiled from source and additionally requires
Clang/libclang and Kerberos development packages, installed by its build helper.
Lean refers to the tool selection, not a minimal disk image.

Like `--tool`, explicit tool selections override method skip flags. Remove a
name from `TOOLS` to omit it. `SKIP_HEAVY=true` documents the intended footprint;
the explicit list is what keeps this profile small.

## Selected tools

| Purpose | Tools | Why included |
| --- | --- | --- |
| Network discovery and access | `nmap`, `dnsutils`, `smbclient`, `netcat-openbsd`, `proxychains4`, `tcpdump` | Service discovery, DNS, SMB access, connections and packet inspection |
| Tunneling | `ligolo-agent`, `ligolo-proxy` | One tunneling toolkit instead of several overlapping choices |
| Windows protocols | `impacket`, `netexec`, `evil-winrm` | Windows protocol utilities, service assessment and WinRM access |
| AD enumeration | `ldapdomaindump`, `enum4linux-ng`, `kerbrute` | LDAP, SMB and Kerberos enumeration |
| BloodHound collection | `rusthound-ce` | Collect AD graph data for BloodHound CE |
| AD permissions and certificates | `bloodyad`, `certipy-ad` | AD object/ACL and AD CS assessment |
| Name resolution assessment | `Responder` | Evaluate local name resolution and authentication exposure |
| C2 | `sliver-client`, `sliver-server` | Sliver operator client and C2 server binaries |
| Web discovery | `subfinder`, `httpx` | Subdomains and live HTTP services |
| Web testing | `ffuf`, `nuclei`, `sqlmap`, `whatweb`, `mitmproxy`, `jwt_tool` | Content discovery, templates, SQL injection, fingerprints, HTTP proxying and JWT inspection |
| Local enumeration resources | `PEASS-ng` | Privilege enumeration resources, including Windows material |

NetExec's command is `nxc`; ProjectDiscovery's `httpx` is the intended binary.
Avoid confusing it with the Python HTTP client of the same name.

## BloodHound setup

`rusthound-ce` is a **collector**, not the BloodHound server/dashboard.
Deploy the dashboard separately, ideally on a dedicated VM, following the
[BloodHound CE installation guide](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart).
The profile deliberately does not pull the BloodHound Docker image: an image
alone is not a configured application with its databases and credentials.

The registry's pipx package `bloodhound` is the legacy Python collector;
it is not a substitute for a CE-compatible collection workflow. RustHound CE
is selected here to keep that distinction explicit. Its upstream source is
[RustHound CE](https://github.com/g0h4n/RustHound-CE).

## Optional additions

- [Burp Suite Community](https://portswigger.net/burp/communitydownload) for
  interactive browser-based web testing. Install separately; it is outside this
  profile's registry selection. `mitmproxy` provides the included HTTP proxy.
- `SecLists` when larger wordlists are needed; omitted to save disk space.
  Install with `sudo bash install.sh --tool SecLists`.
- Windows-native collectors and binaries as the engagement requires. Linux
  Ligolo binaries and cloned PEASS resources do not provision a complete set
  of Windows executables. Select compatible upstream release assets separately.

No tools are installed merely by adding this profile to the repository.
Installation availability and build success depend on the distro/architecture;
Termux skips the RustHound source build and Linux release binaries, so it is not
the recommended platform for this profile.
