# Dev Bootstrap

[![License](https://img.shields.io/badge/license-Proprietary-red.svg)](LICENSE)
[![Python Version](https://img.shields.io/badge/python-3.6%2B-blue.svg)](https://www.python.org/downloads/)

**Eliminating daunting tasks since 2021**

Dev Bootstrap is an automated development environment setup tool that provides fully configured [development containers](https://containers.dev/) for software development. It eliminates the tedious process of setting up development environments by automating the cloning, configuration, and IDE setup for multi-project software solutions.

## Features

- 🚀 **Automated Setup**: One command to clone and configure all projects in a solution
- 🐳 **DevContainer Support**: Pre-configured development containers with all necessary tools
- 🔧 **IDE Integration**: Automatic setup for VSCode and JetBrains IDEs
- 📦 **Multi-Project Management**: Handle solutions with multiple interconnected projects
- 🔀 **Version Control**: Support for specific tags/branches across all projects
- 🎨 **Customizable**: Project-specific, solution-specific, and generic settings layers
- 💾 **Safe Updates**: Automatic backup of existing configurations
- 🐚 **Shell Integration**: Optional mounting of local shell configurations

## DevContainer Features

In addition to automated solution setup, this repository publishes a collection of reusable [DevContainer Features](https://containers.dev/implementors/features/) that can be used in any development container. These features are published to `docker.tern.is/devcontainers/features/` and provide modular functionality for enhancing development environments.

| Feature Name | Reference |
|---|---|
| [Active MQ Build Dependencies](devcontainer-features/src/build-dep-activemq/README.md) | `docker.tern.is/devcontainers/features/build-dep-activemq` |
| [Ansible](devcontainer-features/src/dev-tools-ansible/README.md) | `docker.tern.is/devcontainers/features/dev-tools-ansible` |
| [Audio Build Dependencies](devcontainer-features/src/build-dep-audio/README.md) | `docker.tern.is/devcontainers/features/build-dep-audio` |
| [Audio Setup](devcontainer-features/src/runtime-dep-audio-setup/README.md) | `docker.tern.is/devcontainers/features/runtime-dep-audio-setup` |
| [Authentication Build Dependencies](devcontainer-features/src/build-dep-authentication/README.md) | `docker.tern.is/devcontainers/features/build-dep-authentication` |
| [Boca Printer Driver](devcontainer-features/src/runtime-dep-boca-printer-driver/README.md) | `docker.tern.is/devcontainers/features/runtime-dep-boca-printer-driver` |
| [C++](devcontainer-features/src/dev-tools-cpp/README.md) | `docker.tern.is/devcontainers/features/dev-tools-cpp` |
| [C++ REST SDK Build Dependencies](devcontainer-features/src/build-dep-cpprest/README.md) | `docker.tern.is/devcontainers/features/build-dep-cpprest` |
| [CI/CD Requirements](devcontainer-features/src/cicd-requirements/README.md) | `docker.tern.is/devcontainers/features/cicd-requirements` |
| [Code Linters and Formatters](devcontainer-features/src/dev-tools-linters/README.md) | `docker.tern.is/devcontainers/features/dev-tools-linters` |
| [Configure Tern RPM repositories](devcontainer-features/src/build-dep-setup-tern-repos/README.md) | `docker.tern.is/devcontainers/features/build-dep-setup-tern-repos` |
| [CUPS](devcontainer-features/src/dev-tools-cups/README.md) | `docker.tern.is/devcontainers/features/dev-tools-cups` |
| [Dev Scripts](devcontainer-features/src/devx-dev-scripts/README.md) | `docker.tern.is/devcontainers/features/devx-dev-scripts` |
| [Docker From Docker Support](devcontainer-features/src/devx-docker-outside-of-docker/README.md) | `docker.tern.is/devcontainers/features/devx-docker-outside-of-docker` |
| [Documentation Tools](devcontainer-features/src/dev-tools-documentation/README.md) | `docker.tern.is/devcontainers/features/dev-tools-documentation` |
| [Essential Development Tools and Libraries](devcontainer-features/src/dev-tools-essentials/README.md) | `docker.tern.is/devcontainers/features/dev-tools-essentials` |
| [Font Development Tools](devcontainer-features/src/dev-tools-fonts/README.md) | `docker.tern.is/devcontainers/features/dev-tools-fonts` |
| [GitHub Copilot CLI](devcontainer-features/src/dev-tools-copilot-cli/README.md) | `docker.tern.is/devcontainers/features/dev-tools-copilot-cli` |
| [GNOME Development Tools](devcontainer-features/src/dev-tools-gnome/README.md) | `docker.tern.is/devcontainers/features/dev-tools-gnome` |
| [ISO Creation Tools](devcontainer-features/src/dev-tools-iso/README.md) | `docker.tern.is/devcontainers/features/dev-tools-iso` |
| [JetBrains IDE Support](devcontainer-features/src/devx-jetbrains-ide-support/README.md) | `docker.tern.is/devcontainers/features/devx-jetbrains-ide-support` |
| [JSON Build Dependencies](devcontainer-features/src/build-dep-json/README.md) | `docker.tern.is/devcontainers/features/build-dep-json` |
| [Just Command Runner](devcontainer-features/src/dev-tools-just/README.md) | `docker.tern.is/devcontainers/features/dev-tools-just` |
| [MAG Dev Scripts](devcontainer-features/src/devx-mag-scripts/README.md) | `docker.tern.is/devcontainers/features/devx-mag-scripts` |
| [MariaDB Build Dependencies](devcontainer-features/src/build-dep-mariadb/README.md) | `docker.tern.is/devcontainers/features/build-dep-mariadb` |
| [Mesa D3D12](devcontainer-features/src/devx-mesa-d3d12/README.md) | `docker.tern.is/devcontainers/features/devx-mesa-d3d12` |
| [Multi Service Supervisor](devcontainer-features/src/devx-multi-service-supervisor/README.md) | `docker.tern.is/devcontainers/features/devx-multi-service-supervisor` |
| [Oh My Zsh](devcontainer-features/src/devx-oh-my-zsh/README.md) | `docker.tern.is/devcontainers/features/devx-oh-my-zsh` |
| [OMZ Dev & MAG Plugins](devcontainer-features/src/devx-omz-plugins/README.md) | `docker.tern.is/devcontainers/features/devx-omz-plugins` |
| [OpenCode AI Coding Assistant](devcontainer-features/src/dev-tools-opencode/README.md) | `docker.tern.is/devcontainers/features/dev-tools-opencode` |
| [OpenGL Build Dependencies](devcontainer-features/src/build-dep-opengl/README.md) | `docker.tern.is/devcontainers/features/build-dep-opengl` |
| [Pandoc](devcontainer-features/src/dev-tools-pandoc/README.md) | `docker.tern.is/devcontainers/features/dev-tools-pandoc` |
| [Protocol Buffers Build Dependencies](devcontainer-features/src/build-dep-protobuf/README.md) | `docker.tern.is/devcontainers/features/build-dep-protobuf` |
| [Python Development Tools](devcontainer-features/src/dev-tools-python/README.md) | `docker.tern.is/devcontainers/features/dev-tools-python` |
| [Qt3 Build Dependencies](devcontainer-features/src/build-dep-qt3/README.md) | `docker.tern.is/devcontainers/features/build-dep-qt3` |
| [Qt5 Build Dependencies](devcontainer-features/src/build-dep-qt5/README.md) | `docker.tern.is/devcontainers/features/build-dep-qt5` |
| [Qt6 Build Dependencies](devcontainer-features/src/build-dep-qt6/README.md) | `docker.tern.is/devcontainers/features/build-dep-qt6` |
| [RPM Development Tools](devcontainer-features/src/dev-tools-rpm/README.md) | `docker.tern.is/devcontainers/features/dev-tools-rpm` |
| [Situation Display Map Data](devcontainer-features/src/runtime-dep-situation-display-map-data/README.md) | `docker.tern.is/devcontainers/features/runtime-dep-situation-display-map-data` |
| [SNMP Build Dependencies](devcontainer-features/src/build-dep-snmp/README.md) | `docker.tern.is/devcontainers/features/build-dep-snmp` |
| [SonarQube Development Tools](devcontainer-features/src/dev-tools-sonarqube/README.md) | `docker.tern.is/devcontainers/features/dev-tools-sonarqube` |
| [SQLite Build Dependencies](devcontainer-features/src/build-dep-sqlite/README.md) | `docker.tern.is/devcontainers/features/build-dep-sqlite` |
| [Synth Shell feature](devcontainer-features/src/devx-synth-shell-prompt/README.md) | `docker.tern.is/devcontainers/features/devx-synth-shell-prompt` |
| [Tern Documentation Generation Tools](devcontainer-features/src/dev-tools-tern-documentation-generation/README.md) | `docker.tern.is/devcontainers/features/dev-tools-tern-documentation-generation` |
| [Tern product libraries](devcontainer-features/src/build-dep-ternlibs/README.md) | `docker.tern.is/devcontainers/features/build-dep-ternlibs` |
| [User Setup](devcontainer-features/src/devx-user-setup/README.md) | `docker.tern.is/devcontainers/features/devx-user-setup` |
| [Wanpipe Build Dependencies](devcontainer-features/src/build-dep-wanpipe/README.md) | `docker.tern.is/devcontainers/features/build-dep-wanpipe` |
| [X.Org Build Dependencies](devcontainer-features/src/build-dep-xorg/README.md) | `docker.tern.is/devcontainers/features/build-dep-xorg` |
| [XML Build Dependencies](devcontainer-features/src/build-dep-xml/README.md) | `docker.tern.is/devcontainers/features/build-dep-xml` |

### Using DevContainer Features

Features can be added to any `devcontainer.json` file:

```json
{
  "name": "My Dev Container",
  "image": "mcr.microsoft.com/devcontainers/base:rocky8",
  "features": {
    "docker.tern.is/devcontainers/features/devx-user-setup": {
      "userToCreate": "developer",
      "uidOfUserToCreate": "1000"
    },
    "docker.tern.is/devcontainers/features/devx-docker-outside-of-docker": {},
    "docker.tern.is/devcontainers/features/devx-synth-shell-prompt": {
      "synthPromptUser": "developer"
    }
  }
}
```

For detailed configuration options and usage information, see the README.md file in each feature's directory.

## Quick Start

### Prerequisites

- Python 3.6 or higher
- Git configured with repository access
- Docker (for DevContainer support)
- VSCode or JetBrains IDE with DevContainer support

### Installation

Clone the repository directly and run install:

```bash
git clone -b tasks/devbootstrap_2_0 git@gitlab.com:TernDev/common/dev-bootstrap.git ~/.devbootstrap && ~/.devbootstrap/install.sh
```

The installer clones the repository to `~/.devbootstrap/`, creates a dedicated Python virtual environment at `~/.devbootstrap/.venv/`, and installs a `devbootstrap` shim to `~/.local/bin/`.

Verify the installation:

```bash
devbootstrap -v
```

**Note**: The executable is installed to `~/.local/bin`, which must be in your PATH. Add this to your shell configuration if needed:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

#### Updating

Update to the latest version:

```bash
devbootstrap -u
```

This pulls the latest changes from the repository and re-syncs dependencies.

#### Developer Workflow

Since devbootstrap runs directly from the cloned repository, any edits to
files in `~/.devbootstrap/` take effect immediately — no build step required.
To develop on a separate branch, simply `cd ~/.devbootstrap && git checkout my-branch`.


## Usage

### Basic Usage

Run Dev Bootstrap in interactive mode (recommended for first-time users):

```bash
devbootstrap -i
```

Set up a specific solution:

```bash
devbootstrap -s polaris-mag
```

### Common Options

```bash
# Setup with custom destination directory
devbootstrap -s orion-keflavik -d /path/to/workspace

# Clone only specific projects
devbootstrap -s polaris-mag -p tern-framework tsim

# Disable backups of existing settings
devbootstrap -s tas-keflavik -n

# Use local shell configuration (.bashrc, .zshrc, etc.)
devbootstrap -s polaris-mag -c

# Checkout specific tag/branch
devbootstrap -s polaris-mag -t 25.2.54

# Enable verbose output
devbootstrap -s polaris-mag --verbose

# Get help
devbootstrap --help
```

### Workflow Example

1. **Setup a new solution workspace**:
   ```bash
   devbootstrap -s polaris-mag
   ```
   This creates a `git.polaris-mag/` directory with all required projects.

2. **Open in VSCode**:
   - Open the generated `.code-workspace` file in VSCode
   - VSCode will detect DevContainers and prompt to reopen in container

3. **Start developing**:
   - All tools, dependencies, and configurations are pre-installed
   - Debug configurations, tasks, and settings are ready to use

### System Requirements

#### Required
- **OS**: Linux or Windows with WSL2
- **Git**: Configured with GitLab access
- **Python**: Version 3.6 or higher
- **Docker**: For DevContainer support
- **IDE**: VSCode or JetBrains IDE with DevContainer plugin

#### Optional
- **NVIDIA Container Toolkit**: For [GPU acceleration in Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

### Best Practices

- **Use separate workspace directories** for different solutions (e.g., `git.polaris-mag/`, `git.tas-keflavik/`)
- **Use tags** (`-t` flag) for reproducible environments in production
- **Avoid using local shell configs and stick with the defaults** ( `-c` flag) to prevent conflicts and ensure consistency across environments

## Documentation

- **[User Guide](docs/user-guide.md)**: Detailed usage instructions and examples
- **[Configuration Guide](docs/configuration.md)**: How to configure solutions and projects  
- **[FAQ](docs/faq.md)**: Common questions and troubleshooting
- **[ARCHITECTURE.md](ARCHITECTURE.md)**: Technical architecture and design decisions
- **[CONTRIBUTING.md](CONTRIBUTING.md)**: Guide for contributors

## Supported Solutions

Dev Bootstrap supports the following pre-configured solutions:

- **aries** - ARIES solution configuration
- **nexus-mag** - Nexus MAG solution
- **orion-keflavik** - Orion Keflavik solution
- **polaris-aware** - Polaris AWARE solution
- **polaris-bird** - Polaris BIRD solution
- **polaris-ensure** - Polaris ENSURE solution
- **polaris-ice** - Polaris ICE solution
- **polaris-kos** - Polaris KOS solution
- **polaris-mag** - Polaris MAG solution
- **tas-keflavik** - TAS Keflavik solution
- **terntatoo** - Tern TaToo solution

Run `devbootstrap -i` to see the full list with descriptions in interactive mode.

## Updating

Update to the latest version:

```bash
devbootstrap -u
```

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Architecture

For details on how Dev Bootstrap works — including the configuration system, layered settings, and template variables — see [ARCHITECTURE.md](ARCHITECTURE.md).

## License

Proprietary - Tern Systems

## Support

For issues, questions, or contributions, please contact the Tern Systems development team.
