
# Phantom Framework

**Serverless command-and-control framework built for professional red team operations.**

Cloud functions act as stateless relays. The teamserver never exposes a port to the internet. This is continually being designed from an OPSEC-first principle and as a competitor to Cobalt Strike.<br>

</div>

<br>

## Architecture

Phantom separates the attack surface from the operator. Agents communicate exclusively with serverless cloud functions that relay encrypted data through managed databases. The teamserver polls the database from your internal network. No listening ports, no static infrastructure, no single point of failure.

For air-gapped, on-prem, and classified environments, the standalone **Phantom Listener** replaces cloud infrastructure. A single static binary acts as the HTTPS relay with a local DB. Same wire protocol, same agent payloads, zero external dependencies.

<br>

## Agents

Two native implants. No interpreted languages, no managed runtimes, no framework dependencies.

| | Eclipse | Spectre |
|:--|:--|:--|
| **Platform** | Windows | Linux |
| **Language** | C++ / x64 NASM | C |
| **Output** | PIC shellcode, EXE, DLL, Service EXE | ELF, raw binary |
| **Linking** | Fully position-independent, zero static imports | PIE with optional musl static linking |
| **BOF engine** | COFF loader with Beacon API compatibility | ELF loader with trampoline table resolution |

These are the built-ins. Any other beacon - your own implant, a team-specific loader, a research agent - slots in as an external process through the plugin SDK (see below) without touching the teamserver.

<br>

---

<br>

## Capability Overview

<table>
<thead>
<tr>
<th width="220"><sub>CAPABILITY</sub></th>
<th width="440"><sub>PHANTOM C2</sub></th>
<th width="340"><sub>OTHER VENDORS</sub></th>
</tr>
</thead>
<tbody>

<tr>
<td><b>SERVERLESS INFRASTRUCTURE</b></td>
<td>
<ul>
<li>Zero listening ports. Agents talk to cloud functions, not your server.</li>
<li>Native support for AWS, Azure, and GCP with full Terraform IaC.</li>
<li>Infrastructure is ephemeral by design. Tear down and redeploy in minutes.</li>
</ul>
</td>
<td>
<ul>
<li>Most frameworks require a publicly exposed listener (HTTP/S, DNS, SMB) that can be fingerprinted and blocked.</li>
</ul>
</td>
</tr>

<tr>
<td><b>AIR-GAPPED / ON-PREM OPERATIONS</b></td>
<td>
<ul>
<li>Standalone HTTPS listener binary. Single binary with embedded database, no runtime dependencies, no cloud access required.</li>
<li>Drop-in replacement for cloud relays. Agents use the same wire protocol, same payload builds, same relay authentication.</li>
<li>Deploy to air-gapped networks, classified environments, on-prem labs, or DMZ hosts where cloud infrastructure is not permitted.</li>
<li>Hybrid deployments: mix cloud relays and listeners in the same agent relay pool for automatic failover across infrastructure types.</li>
</ul>
</td>
<td>
<ul>
<li>Most frameworks require either a cloud backend or a traditional listener with exposed ports. Operating in disconnected environments without redesigning the C2 chain is rarely supported.</li>
</ul>
</td>
</tr>

<tr>
<td><b>CDN DOMAIN FRONTING</b></td>
<td>
<ul>
<li>First-class CDN relay support across all three cloud providers.</li>
<li>Agents connect through CloudFront, Azure Front Door, or Cloud CDN. Traffic appears as requests to high-reputation cloud domains.</li>
<li>Optional custom domain with managed TLS certificates.</li>
</ul>
</td>
<td>
<ul>
<li>Domain fronting support is often manual, fragile, or limited to a single CDN provider.</li>
</ul>
</td>
</tr>

<tr>
<td><b>AGENT ARCHITECTURE</b></td>
<td>
<ul>
<li>Fully position-independent shellcode agent with zero static imports.</li>
<li>All Win32 APIs resolved at runtime. Zero static imports in the import table.</li>
<li>Native Linux agent with optional static linking for zero-dependency deployment.</li>
</ul>
</td>
<td>
<ul>
<li>Commonly depend on reflective DLL loading or managed runtimes with larger footprints and more observable loading behavior.</li>
</ul>
</td>
</tr>

<tr>
<td><b>POLYMORPHIC LOADER</b></td>
<td>
<ul>
<li>Every build produces a unique binary. Randomized function names, variable names, hash seeds, and junk code insertion.</li>
<li>Multi-layer string obfuscation with randomized encoding per build.</li>
<li>Shellcode encryption with a fresh key per build.</li>
</ul>
</td>
<td>
<ul>
<li>Operators often rely on external packers, manual obfuscation, or static loader templates that share signatures across deployments.</li>
</ul>
</td>
</tr>

<tr>
<td><b>MODULE STOMPING</b></td>
<td>
<ul>
<li>Shellcode is planted into a legitimate module's memory range rather than privately allocated regions.</li>
<li>Execution originates from a signed module, blending into normal process memory layout.</li>
</ul>
</td>
<td>
<ul>
<li>Shellcode typically runs from privately allocated RWX memory, which is more visible to memory scanners and ETW telemetry.</li>
</ul>
</td>
</tr>

<tr>
<td><b>SLEEP OBFUSCATION</b></td>
<td>
<ul>
<li>Sleep obfuscation with configurable masking modes.</li>
<li>In-memory content is encrypted while the agent sleeps.</li>
</ul>
</td>
<td>
<ul>
<li>Often based on public x64-only sleep mask techniques that are well-signatured.</li>
</ul>
</td>
</tr>

<tr>
<td><b>STACK SPOOFING</b></td>
<td>
<ul>
<li>Full call stack frame forgery that presents a clean, legitimate-looking call chain.</li>
<li>Applied to every API call, not just sleep.</li>
</ul>
</td>
<td>
<ul>
<li>Stack spoofing is often limited to sleep calls or a narrow set of APIs, leaving other calls exposed to stack-walking detections.</li>
</ul>
</td>
</tr>

<tr>
<td><b>DIRECT &amp; INDIRECT SYSCALLS</b></td>
<td>
<ul>
<li>Three configurable modes: disabled, direct, or indirect.</li>
<li>Covers critical NT APIs. Bypasses userland hooks entirely.</li>
</ul>
</td>
<td>
<ul>
<li>Syscall support is frequently all-or-nothing with no operator control over the mode, or relies on public implementations that are well-known to EDR vendors.</li>
</ul>
</td>
</tr>

<tr>
<td><b>AMSI / ETW BYPASS</b></td>
<td>
<ul>
<li>Built-in bypass for both AMSI and ETW-TI.</li>
<li>Separate bypass for .NET assembly execution contexts.</li>
</ul>
</td>
<td>
<ul>
<li>Runtime patching approaches are increasingly detected. Some frameworks omit ETW-TI bypass entirely.</li>
</ul>
</td>
</tr>

<tr>
<td><b>MALLEABLE PROFILES</b></td>
<td>
<ul>
<li>JSON-based malleable HTTP profiles with per-URI client and server output configuration.</li>
<li>Output placement: body, header, cookie, or parameter.</li>
<li>Data transforms: XOR mask, prepend, append, base64, base64url, base32, hex.</li>
</ul>
</td>
<td>
<ul>
<li>Profile systems exist but are often limited in transform depth or lack per-URI granularity.</li>
</ul>
</td>
</tr>

<tr>
<td><b>RELAY RESILIENCE</b></td>
<td>
<ul>
<li>Up to 10 relay URLs per agent with three rotation strategies: random, round-robin, and failover with automatic fallback.</li>
<li>Relay pool is embedded at build time. The agent rotates through the pool on every callback.</li>
<li>Relay authentication prevents unauthorized interaction with deployed infrastructure.</li>
<li>Stateless relay design means any function instance can serve any request. No sticky sessions, no affinity.</li>
<li>Encrypted transit: both agents callback over HTTPS with TLS, and every relay request is signed with the agent's per-build relay key.</li>
</ul>
</td>
<td>
<ul>
<li>Callback resilience is typically limited to a small number of fallback URLs with no rotation strategy control.</li>
</ul>
</td>
</tr>

<tr>
<td><b>VIRTUAL MACHINE EXECUTION</b></td>
<td>
<ul>
<li>Custom bytecode RISC-V VM with a full instruction set and native function interop.</li>
<li>Server-side assembler compiles VM programs for deployment to agents.</li>
<li>Beacon API integration: output, token manipulation, and file downloads from VM programs.</li>
</ul>
</td>
<td>
<ul>
<li>Not commonly available. Post-exploitation logic typically runs as shellcode or BOFs with no abstraction layer.</li>
</ul>
</td>
</tr>

<tr>
<td><b>BOF EXECUTION</b></td>
<td>
<ul>
<li>Windows: COFF loader with full Beacon API compatibility and Async execution.</li>
<li>Linux: ELF64 loader with broad libc symbol resolution.</li>
<li>BOF API calls are routed through the agent's evasion layer.</li>
</ul>
</td>
<td>
<ul>
<li>BOF support is often Windows-only. Linux BOF execution is rare. API proxying through syscalls is not standard.</li>
</ul>
</td>
</tr>

<tr>
<td><b>EXTENSION SYSTEM</b></td>
<td>
<ul>
<li>JavaScript scripting engine (pfScript) with sandboxed execution, import system, and typed command definitions.</li>
<li>Pre-hook argument parsing, BOF/VM parameter packing, and post-hook credential extraction.</li>
<li>Hot-loadable extension packs with platform and agent-type filtering.</li>
<li>Drop-in compatibility with Adaptix-framework pf scripts - established packs such as PostEx-Arsenal run unmodified.</li>
</ul>
</td>
<td>
<ul>
<li>Extension APIs exist but are often tightly coupled to the framework internals or lack runtime loading and typed argument support.</li>
</ul>
</td>
</tr>

<tr>
<td><b>EXTERNAL AGENT PLUGINS (SDK)</b></td>
<td>
<ul>
<li>Integrate any external beacon - or your own payload builder/loader - as a supervised separate process speaking JSON-RPC 2.0 over stdin/stdout. Custom agents integrate without source code access or teamserver recompilation.</li>
<li>Official Go SDK: implement the protocol and/or builder interface and call <code>Serve()</code>. The wire contract is plain JSON, so plugins can be written in any language.</li>
<li>Decide per agent what the plugin owns: check-in protocol, payload builds, or both.</li>
<li>Lifecycle fully managed by the server: handshake and capability negotiation, health pings, crash recovery with restart limits, and per-agent core BOF and module directories passed at initialization.</li>
</ul>
</td>
<td>
<ul>
<li>Adding a new implant family usually means patching framework internals, recompiling the teamserver, and maintaining a fork. Supported agents are limited to the vendor's own implementations.</li>
</ul>
</td>
</tr>

<tr>
<td><b>PROCESS INJECTION</b></td>
<td>
<ul>
<li>Multiple thread pool-based injection variants.</li>
<li>Additional methods including section mapping, CFG-aware injection, and cross-architecture (WoW64) injection.</li>
</ul>
</td>
<td>
<ul>
<li>Injection is commonly limited to classic techniques (APC, thread hijacking, process hollowing) that are well-covered by behavioral detections.</li>
</ul>
</td>
</tr>

<tr>
<td><b>LATERAL MOVEMENT</b></td>
<td>
<ul>
<li>BOF-based PsExec, SCShell, WinRM, WMI, and DCOM execution.</li>
<li>SMB named pipe pivoting for agent chaining.</li>
</ul>
</td>
<td>
<ul>
<li>Lateral movement modules are present but often rely on fork-and-run patterns or external tooling.</li>
</ul>
</td>
</tr>

<tr>
<td><b>PEER-TO-PEER</b></td>
<td>
<ul>
<li>SMB named pipe & TCP pivoting for agent chaining. Parent agents relay traffic for child agents that have no direct internet access.</li>
<li>Link and unlink agents at runtime. The pivot graph is tracked in the UI with parent/child relationships.</li>
<li>Child agents inherit the parent's relay path. No additional infrastructure required to reach segmented networks.</li>
</ul>
</td>
<td>
<ul>
<li>P2P pivoting is available in some frameworks but often limited to a single transport or requires manual configuration. Dynamic link/unlink and integrated pivot tracking are uncommon.</li>
</ul>
</td>
</tr>

<tr>
<td><b>SOCKS &amp; TUNNELING</b></td>
<td>
<ul>
<li>SOCKS4 and SOCKS5 proxy with optional username/password authentication.</li>
<li>Local and reverse port forwarding through the agent.</li>
<li>Per-connection channel multiplexing over the C2 protocol.</li>
<li>Managed from the operator console. Active tunnels are tracked in a dedicated Tunnels panel with badge counts.</li>
</ul>
</td>
<td>
<ul>
<li>SOCKS support is common but often limited to a single protocol version without authentication. Tunnel management is typically CLI-only with no integrated UI.</li>
</ul>
</td>
</tr>

<tr>
<td><b>CREDENTIAL OPERATIONS</b></td>
<td>
<ul>
<li>Token manipulations, LSASS dump, DCSync with automatic credential parsing, SAM hashdump, LAPS retrieval.</li>
<li>Credential store with automatic extraction from BOF output. Parsed results are immediately available to the operator.</li>
</ul>
</td>
<td>
<ul>
<li>Credential extraction often requires manual parsing of tool output. Integrated credential stores are uncommon.</li>
</ul>
</td>
</tr>

<tr>
<td><b>PRIVILEGE ESCALATION</b></td>
<td>
<ul>
<li>Token impersonation to SYSTEM and TrustedInstaller.</li>
<li>Multiple UAC bypass techniques (SSPI Datagram, Registry Shell Command).</li>
<li>DCOMPotato for SeImpersonate to SYSTEM.</li>
</ul>
</td>
<td>
<ul>
<li>Elevation paths are available but often depend on external binaries or well-signatured public tools.</li>
</ul>
</td>
</tr>

<tr>
<td><b>ACTIVE DIRECTORY</b></td>
<td>
<ul>
<li>Credential extraction results are automatically parsed and fed into the framework's credential store. DCSync hashes, LAPS passwords, and Kerberos keys appear in the operator console immediately.</li>
<li>Enumerated hosts are auto-populated into the Targets panel for tracking across the engagement.</li>
<li>Integrated relay viability checks across LDAP, SMB, HTTP, and MSSQL protocols.</li>
</ul>
</td>
<td>
<ul>
<li>Credential output usually requires manual parsing, and results are not fed back into the framework automatically.</li>
</ul>
</td>
</tr>

<tr>
<td><b>CLOUD PAYLOAD HOSTING</b></td>
<td>
<ul>
<li>Upload and serve payloads via Stateless relays directly from the operator console.</li>
<li>Host and unhost payloads on demand. Cloud storage lifecycle is fully managed by the teamserver.</li>
</ul>
</td>
<td>
<ul>
<li>Payload hosting is typically manual. Operators stage files on infrastructure they manage separately from the C2 framework.</li>
</ul>
</td>
</tr>

<tr>
<td><b>.NET &amp; PE EXECUTION</b></td>
<td>
<ul>
<li>In-process or Fork-and-run .NET assembly execution (<code>execute-assembly</code>) with synchronous and async modes.</li>
<li>Unmanaged PE execution in agent memory with support for EXE and DLL binaries, method selection, PE caching, and dependency loading.</li>
<li>AMSI and ETW bypass applied to .NET execution contexts.</li>
</ul>
</td>
<td>
<ul>
<li>.NET execution is common but frequently uses fork-and-run, spawning a sacrificial process per assembly. In-memory PE loading with caching and dependency handling is rare.</li>
</ul>
</td>
</tr>

<tr>
<td><b>OPERATOR CONSOLE</b></td>
<td>
<ul>
<li>Real-time web UI over WebSocket. Session table, network graph, and global task views. Tabbed lower pane with drag-and-drop reordering and side-by-side split.</li>
<li>Dock layout selection support.</li>
<li>Per-agent console, file browser, process browser, and screenshot gallery.</li>
<li>Integrated credential store with search, export, and automatic population from post-exploitation output.</li>
<li>Payload store, tunnel manager, target tracker, team chat, and event log.</li>
<li>Built-in pf script editor, VM bytecode editor, and communication profile editor.</li>
<li>Multi-operator presence: see who is online, which agent they are focused on, and when they are typing.</li>
<li>Role-based access control with admin and operator roles. Read-only mode for observers.</li>
<li>One-click operation report export as a self-contained PDF document. All operational data is mapped to MITRE ATT&amp;CK tactics and techniques. Includes tactic/technique distribution charts, IOC extraction, and a filterable chronological timeline.</li>
</ul>
</td>
<td>
<ul>
<li>Most frameworks provide a CLI or basic web UI. Integrated editors, multi-operator presence, and automatic credential/target population from tool output are uncommon.</li>
</ul>
</td>
</tr>

<tr>
<td><b>OPERATIONAL GUARDRAILS</b></td>
<td>
<ul>
<li>Target validation by username, hostname, IP address, and domain.</li>
<li>Kill date with automatic agent self-termination.</li>
<li>Worktime restrictions to limit activity to defined hours.</li>
</ul>
</td>
<td>
<ul>
<li>Guardrails are often limited to kill dates. Target-aware execution constraints are uncommon.</li>
</ul>
</td>
</tr>

<tr>
<td><b>SECURITY TELEMETRY</b></td>
<td>
<ul>
<li>Agents report target security posture on check-in: CFG, HVCI, DSE, Secure Boot, test signing, debug mode.</li>
<li>Operators see the defensive landscape before tasking.</li>
</ul>
</td>
<td>
<ul>
<li>Security feature detection is usually a manual post-exploitation step rather than an automatic check-in property.</li>
</ul>
</td>
</tr>

</tbody>
</table>

<br>

## Contact

For licensing, demonstrations, or technical inquiries, reach out to the Phantom Framework team.

