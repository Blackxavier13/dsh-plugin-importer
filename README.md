# DSH Universal Importer

Universal source importer and packager for **DeepSeek Harness (DSH)** environments.

DSH Universal Importer converts skills, MCP configurations, and existing DSH/Cordis plugin bundles from multiple sources into installable DSH plugin packages. It is designed to make third-party agentic capabilities portable across a DSH profile without requiring every source repository to already use the DeepSeek Harness plugin layout.

> **Status:** Early release / developer-oriented tooling
>
> **Important:** DeepSeek Harness and its plugin interfaces may change. Always validate generated bundles against the version of DSH installed on your machine before using third-party packages.

---

## What problem does this solve?

Agentic development ecosystems expose reusable capabilities in many different formats:

- `SKILL.md`-based skills
- Git repositories
- GitHub repositories
- GitHub users/organizations
- ZIP/TAR archives
- HTTP/HTTPS URLs
- local folders
- local files
- MCP server configurations
- existing DSH/Cordis plugins

Without an importer, every capability has to be manually adapted to the host harness.

DSH Universal Importer provides a common pipeline:

```text
                    SOURCE ECOSYSTEMS
 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 │ Git / GitHub │ HTTP / HTTPS │ Local files  │ MCP configs  │
 └──────┬───────┴──────┬───────┴──────┬───────┴──────┬───────┘
        │              │              │              │
        └──────────────┴──────────────┴──────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Source Acquisition   │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Format Detection    │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Normalization       │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ DSH Bundle Builder  │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Validation          │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │ DSH Plugin Manager  │
                    └──────────┬──────────┘
                               ▼
                         DSH PROFILE
```

The importer **does not replace DSH**. It prepares capabilities in a format that DSH can install and load through its normal plugin mechanism.

---

## Core capabilities

### 1. Universal source acquisition

Sources can be supplied as:

| Source | Example |
|---|---|
| Local directory | `C:\AI\my-skill` |
| Local file | `C:\AI\plugin.zip` |
| `file://` URL | `file:///C:/AI/plugin.zip` |
| HTTP URL | `https://example.com/skill.zip` |
| HTTPS URL | `https://example.com/skill.tar.gz` |
| Git repository | `git+https://github.com/user/repo.git` |
| GitHub repository | `github:user/repo` |
| GitHub user | `github-user:user` |

The exact accepted URL/specifier forms are handled by the CLI source resolver.

### 2. Automatic format detection

The importer can inspect a source and identify supported capability types, including:

- Skills
- MCP configurations
- Native DSH/Cordis bundles

This allows:

```powershell
python -m dsh_importer.cli import github:OWNER/REPOSITORY --kind auto
```

instead of requiring the user to know the internal format first.

### 3. Skill conversion

Skill repositories containing recognized skill definitions such as `SKILL.md` can be wrapped in a DSH bundle.

The generated bundle contains the skill resources and a DSH plugin entry point that registers the capability with the Harness skill subsystem.

Conceptually:

```text
SKILL.md
   │
   ▼
Skill detector
   │
   ▼
Skill normalizer
   │
   ▼
DSH skill bundle
   │
   ▼
dsh plugin add
```

### 4. MCP conversion

Recognized MCP configuration files such as `.mcp.json` or `mcp.json` can be converted into DSH-compatible MCP plugin bundles.

The generated plugin uses the DSH MCP client integration rather than implementing an independent MCP runtime.

Supported MCP transport/configuration depends on the installed DSH MCP client version. The current architecture is intended for common `stdio` and streamable HTTP configurations.

### 5. Native DSH plugin preservation

If the source is already a valid DSH/Cordis plugin bundle, the importer does not unnecessarily rewrite it.

This prevents the importer from damaging plugin-specific executable code simply because the package originated outside the importer.

### 6. Validation before installation

Generated packages can be validated before they are handed to DSH.

Recommended workflow:

```text
Import
  ↓
Inspect
  ↓
Validate
  ↓
Review
  ↓
Install
```

This is especially important for third-party repositories.

---

# Installation

## Requirements

Recommended environment:

- Windows, Linux, or macOS
- Python 3.10+
- Git for Git-based sources
- Node.js for DSH-related runtime validation/installation where required
- A working DeepSeek Harness installation for final installation

The importer itself is intentionally kept lightweight.

## Install from source

Clone the repository:

```powershell
git clone https://github.com/YOUR-USERNAME/dsh-universal-importer.git
cd dsh-universal-importer
```

Create a virtual environment if desired:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install the project:

```powershell
python -m pip install -e .
```

Verify the CLI:

```powershell
python -m dsh_importer.cli --help
```

> Replace `YOUR-USERNAME` with the actual GitHub repository owner after publishing this project.

---

# Connecting the importer to DeepSeek Harness

The importer and DSH have deliberately separate responsibilities.

```text
┌────────────────────────────┐
│ DSH Universal Importer     │
│                            │
│ Acquire                    │
│ Detect                     │
│ Normalize                  │
│ Build                      │
│ Validate                   │
└─────────────┬──────────────┘
              │
              │ generated DSH bundle
              ▼
┌─────────────────────────────┐
│ DeepSeek Harness            │
│                             │
│ dsh plugin manager          │
│                             │
│     selected profile        │
└─────────────────────────────┘
```

For example:

```powershell
python -m dsh_importer.cli install C:\AI\dsh-imports\bundles\my-plugin --profile web
```

Conceptually this performs the equivalent of:

```powershell
dsh plugin --profile web add C:\AI\dsh-imports\bundles\my-plugin
```

The generated bundle therefore remains compatible with DSH's normal plugin lifecycle instead of requiring a separate always-running bridge process.

---

# Basic usage

## Inspect a source

Before importing a third-party source:

```powershell
python -m dsh_importer.cli inspect github:OWNER/REPOSITORY
```

Use this to determine what the importer recognizes.

---

## Import a skill

Local skill:

```powershell
python -m dsh_importer.cli import C:\AI\my-skill --kind skill --out C:\AI\dsh-imports
```

GitHub repository:

```powershell
python -m dsh_importer.cli import github:OWNER/REPOSITORY --kind skill --out C:\AI\dsh-imports
```

Automatic detection:

```powershell
python -m dsh_importer.cli import github:OWNER/REPOSITORY --kind auto --out C:\AI\dsh-imports
```

---

## Import an MCP configuration

For example:

```powershell
python -m dsh_importer.cli import C:\AI\my-mcp.json --kind mcp --out C:\AI\dsh-imports
```

The importer creates a DSH plugin bundle containing the MCP configuration and the corresponding DSH MCP integration.

---

## Import an existing DSH plugin

```powershell
python -m dsh_importer.cli import C:\AI\existing-dsh-plugin --kind plugin --out C:\AI\dsh-imports
```

Native DSH bundles are preserved rather than reconstructed unnecessarily.

---

## Import from GitHub

Repository:

```powershell
python -m dsh_importer.cli import github:OWNER/REPOSITORY --kind auto --out C:\AI\dsh-imports
```

GitHub user:

```powershell
python -m dsh_importer.cli import github-user:USERNAME --kind auto --limit 20 --out C:\AI\dsh-imports
```

The GitHub-user mode is useful for discovering several repositories belonging to an individual or organization and importing the recognized capabilities.

---

# Batch importing

Create a source file such as:

```text
# DSH capability sources
github:OWNER/skill-one
github:OWNER/skill-two
https://example.com/plugin.zip
C:\AI\local-skill
github:OWNER/mcp-server
```

Then run:

```powershell
python -m dsh_importer.cli batch C:\AI\sources.txt --out C:\AI\dsh-imports
```

This provides the foundation for maintaining a personal capability catalog.

---

# Validate generated bundles

Always validate before installation:

```powershell
python -m dsh_importer.cli validate C:\AI\dsh-imports
```

Validation checks the generated DSH package structure and relevant metadata before handing it to the Harness plugin manager.

---

# Install into a DSH profile

After validation:

```powershell
python -m dsh_importer.cli install C:\AI\dsh-imports\bundles\PLUGIN_NAME --profile web
```

Other DSH profiles can be supplied if supported by your installed Harness version.

The importer intentionally delegates final plugin registration to DSH.

---

# Generated bundle structure

A generated skill plugin is conceptually structured like:

```text
bundles/
└── dsh-imported-skill-example/
    ├── package.json
    ├── cordis.patch.yml
    ├── index.js
    └── skills/
        └── example/
            └── SKILL.md
```

An MCP bundle is conceptually:

```text
bundles/
└── dsh-imported-mcp-example/
    ├── package.json
    ├── cordis.patch.yml
    ├── index.js
    └── mcp/
        └── config.json
```

Exact generated paths may evolve with the importer version.

---

# Architecture

The project is organized around five logical layers.

```text
                 ┌────────────────────┐
                 │       CLI          │
                 └─────────┬──────────┘
                           │
                 ┌─────────▼──────────┐
                 │ Source Acquisition │
                 └─────────┬──────────┘
                           │
                 ┌─────────▼──────────┐
                 │ Format Detection   │
                 └─────────┬──────────┘
                           │
                 ┌─────────▼──────────┐
                 │ Capability Builder │
                 └─────────┬──────────┘
                           │
                 ┌─────────▼──────────┐
                 │ DSH Bundle Builder │
                 └─────────┬──────────┘
                           │
                 ┌─────────▼──────────┐
                 │ Validator/Installer│
                 └────────────────────┘
```

## Source acquisition

Responsible for obtaining source material from:

- filesystem
- URLs
- Git
- GitHub
- archives

## Format detection

Determines whether a source contains:

- one or more skills
- MCP configuration
- native DSH plugin metadata
- unsupported content

## Capability normalization

Converts different source layouts into a common internal representation.

For example:

```text
Git repository
       │
       ├── SKILL.md
       ├── README.md
       └── examples/
              │
              ▼
       normalized skill
```

## Bundle generation

The normalized representation is compiled into the DSH/Cordis plugin structure.

## Installation

The generated bundle is passed to the normal DSH plugin manager.

---

# Security model

Importing third-party agentic capabilities is inherently security-sensitive.

A skill, MCP server, or DSH plugin may contain code capable of accessing:

- local files
- environment variables
- subprocesses
- network resources
- credentials
- development tools
- other MCP servers

Therefore:

> **Importing a package does not mean that the package is trustworthy.**

The importer is a packaging and interoperability tool, not a security certification system.

## Recommended workflow

```text
Unknown source
     ↓
inspect
     ↓
review source code/configuration
     ↓
validate generated bundle
     ↓
install in isolated profile
     ↓
test
     ↓
use in normal workflow
```

For untrusted sources, prefer an isolated environment and do not expose unnecessary credentials.

---

# Secret handling

MCP configurations sometimes contain values such as:

```json
{
  "env": {
    "GITHUB_TOKEN": "..."
  }
}
```

The importer is designed to avoid unnecessarily embedding secret values into generated plugin configuration. Secret-like values should instead be represented through environment variables where supported.

For example:

```text
GITHUB_TOKEN=your-secret
```

and the generated configuration references the environment instead of committing the secret into the generated bundle.

**Never commit API keys, tokens, passwords, private certificates, or other credentials to this repository.**

---

# Why arbitrary repositories are not blindly converted

Not every agent repository can safely or correctly become a DSH plugin.

For example:

```text
random Python application
```

is not automatically equivalent to:

```text
DSH plugin
```

A correct conversion may require understanding:

- lifecycle hooks
- plugin context
- runtime dependencies
- prompts
- tool contracts
- MCP transport
- filesystem assumptions
- environment variables
- process management

The importer therefore uses a conservative approach:

1. Detect known formats.
2. Preserve native DSH bundles.
3. Convert recognized skills.
4. Convert recognized MCP configurations.
5. Reject or report unsupported layouts instead of pretending they are valid DSH plugins.

---

# Extending the importer

The intended long-term architecture supports additional source adapters and capability compilers.

Potential future adapters include:

```text
GitHub
GitLab
Bitbucket
Hugging Face
npm packages
PyPI packages
URL catalogs
local folders
local archives
plugin registries
```

Potential capability types include:

```text
Skills
MCP servers
DSH plugins
Agents
Commands
Prompts
Workflows
Hooks
Tool definitions
```

A new importer should follow this general pipeline:

```text
source adapter
      ↓
source manifest
      ↓
normalized capability
      ↓
capability compiler
      ↓
DSH bundle
      ↓
validator
```

This prevents source-specific logic from leaking into the DSH bundle generator.

---

# Building a personal capability registry

The importer can also serve as the foundation of a personal registry.

For example:

```text
C:\AI\dsh-registry\
│
├── sources.txt
├── skills\
├── mcp\
├── plugins\
├── bundles\
└── manifests\
```

A registry could track:

- source URL
- source type
- repository revision
- imported version
- capability type
- generated bundle
- validation status
- installation profile
- trust level

This makes it possible to move toward reproducible DSH environments rather than manually installing plugins one at a time.

---

# Example end-to-end workflow

## 1. Discover

```powershell
python -m dsh_importer.cli inspect github:OWNER/REPOSITORY
```

## 2. Import

```powershell
python -m dsh_importer.cli import github:OWNER/REPOSITORY --kind auto --out C:\AI\dsh-imports
```

## 3. Validate

```powershell
python -m dsh_importer.cli validate C:\AI\dsh-imports
```

## 4. Review generated files

Inspect:

```text
package.json
cordis.patch.yml
index.js
skills/
mcp/
```

## 5. Install

```powershell
python -m dsh_importer.cli install C:\AI\dsh-imports\bundles\PLUGIN_NAME --profile web
```

## 6. Test in DSH

Start/use the corresponding DSH profile and verify that the imported skill, MCP server, or plugin is discoverable and behaves as expected.

---

# Troubleshooting

## `dsh` is not recognized

The importer can generate bundles without a working DSH installation, but final installation requires the DSH CLI to be available on `PATH` or otherwise accessible to the installation command.

Verify:

```powershell
dsh --help
```

## Git source cannot be cloned

Check:

```powershell
git --version
```

Then verify that the repository URL is accessible.

For private repositories, authenticate Git using your normal Git credential mechanism rather than putting credentials directly into source URLs.

## A repository is detected as unsupported

Run:

```powershell
python -m dsh_importer.cli inspect <SOURCE>
```

Then check whether the repository actually contains a recognized skill, MCP configuration, or native DSH bundle.

If it is an agent/workflow repository with no recognized format, it may require a dedicated compiler/adapter rather than automatic conversion.

## Generated plugin does not load

First validate the generated bundle:

```powershell
python -m dsh_importer.cli validate <OUTPUT>
```

Then inspect the generated `package.json` and `cordis.patch.yml` against the version of DSH installed locally.

DSH plugin APIs can change between releases, so a bundle generated for one Harness version may require adjustment for another.

---

# Development

Run the test suite from the repository root using the project's configured test command.

Recommended checks before submitting a change:

```text
source detection
source acquisition
skill conversion
MCP conversion
native plugin preservation
bundle validation
CLI behavior
Node/plugin syntax
installation integration
```

When adding a new source type, add tests for both successful acquisition and failure cases.

When adding a new capability compiler, test the generated bundle rather than testing only the intermediate Python objects.

---

# Design principles

### 1. DSH remains the runtime

The importer packages capabilities for DSH instead of creating a competing runtime.

### 2. Preserve native packages

If a source already contains a valid DSH plugin, do not unnecessarily rewrite it.

### 3. Fail safely

Unknown structures should be reported instead of being converted incorrectly.

### 4. Keep acquisition separate from compilation

Downloading a repository and understanding its capability are different responsibilities.

### 5. Validate before installation

Generated output should be inspectable and testable before it reaches the Harness.

### 6. Do not hide third-party behavior

The importer should make it easier—not harder—to inspect what a third-party capability contains.

---

# Roadmap

Planned/possible future capabilities:

- [ ] persistent source registry
- [ ] dependency lockfile
- [ ] source revision pinning
- [ ] SHA-256 artifact verification
- [ ] signed plugin verification
- [ ] GitHub release/tag selection
- [ ] GitLab repository support
- [ ] archive URL support improvements
- [ ] skill dependency resolution
- [ ] MCP dependency resolution
- [ ] automatic DSH version compatibility checks
- [ ] dry-run installation
- [ ] rollback/uninstall management
- [ ] plugin update detection
- [ ] registry synchronization
- [ ] capability search/indexing
- [ ] trust policies
- [ ] sandboxed inspection
- [ ] web UI for managing imported capabilities
- [ ] automatic skill discovery/routing metadata

The longer-term goal is to evolve from a **universal importer** into a **DSH capability registry and synchronization layer**.

---

# Relationship to DSH

DSH Universal Importer is complementary to DeepSeek Harness:

```text
              DSH Universal Importer
                       │
       ┌───────────────┼────────────────┐
       │               │                │
     Skills           MCP          DSH Plugins
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                DSH/Cordis Bundle
                       │
                       ▼
               DSH Plugin Manager
                       │
                       ▼
                 DSH Profile
                       │
                       ▼
                 DeepSeek Agent
```

The importer is therefore best understood as a **capability ingestion and packaging layer** for DeepSeek Harness.

---

# Contributing

Contributions are welcome.

Good areas for contribution include:

- new source adapters
- new capability detectors
- new DSH bundle compilers
- validation improvements
- security hardening
- compatibility tests against DSH releases
- documentation
- integration tests

Before adding a large abstraction, prefer a small adapter that solves a demonstrated source format. Keep source-specific behavior isolated from the core bundle model.

---

# License

Add the project's chosen license here before publishing the repository.

For example:

```text
MIT License
```

Do not claim a license that has not actually been added to the repository.

---

# Disclaimer

This project is an independent community/developer tool unless explicitly stated otherwise. It should not be assumed to be an official DeepSeek product.

Third-party skills, MCP servers, repositories, and plugins are supplied by their respective authors. Review and trust them before installation.

---

## Quick reference

```powershell
# Inspect
python -m dsh_importer.cli inspect <SOURCE>

# Import automatically
python -m dsh_importer.cli import <SOURCE> --kind auto --out C:\AI\dsh-imports

# Import a skill
python -m dsh_importer.cli import <SOURCE> --kind skill --out C:\AI\dsh-imports

# Import MCP
python -m dsh_importer.cli import <SOURCE> --kind mcp --out C:\AI\dsh-imports

# Import a native DSH plugin
python -m dsh_importer.cli import <SOURCE> --kind plugin --out C:\AI\dsh-imports

# Import many sources
python -m dsh_importer.cli batch C:\AI\sources.txt --out C:\AI\dsh-imports

# Validate
python -m dsh_importer.cli validate C:\AI\dsh-imports

# Install into DSH
python -m dsh_importer.cli install C:\AI\dsh-imports\bundles\PLUGIN_NAME --profile web
```
