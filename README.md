# Ansible Role: Apache Tomcat App

| Source                                                                                                                                  | Version                                                                                                                                                                    | CI                                                                                                                                                                                                  | License                                                                                          |
| --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app) | [![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-apache-tomcat-app)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/releases) | [![CI](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat-app/actions/workflows/ci.yml) | [![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE) |

This Ansible role manages **Apache Tomcat application contexts** and configuration. Enterprise companion to the [apache_tomcat](../ansible-role-apache-tomcat/) role, it handles everything related to application deployment configuration except the WAR/JAR file itself: context XML files, JNDI DataSources, externalized config files (Spring Boot `application.properties`), Environment entries, Resource Links, Context Parameters, and optional JDBC driver installation.

## ✨ Features

- 🔧 **Context XML Management** — Deploys templated context XML files per application to `conf/Catalina/localhost/`
- 🗄️ **JNDI DataSources** — Configures database connection pools (`javax.sql.DataSource`) with full tuning parameters (pool size, validation, abandoned connection handling)
- 📁 **Config File Isolation** — Deploys application config files to isolated per-app directories (`CATALINA_BASE/config/<app>/`) with Jinja2 templating
- 🌱 **Spring Boot Integration** — Auto-injects `spring.config.additional-location` Context Parameter for externalized properties
- 🚀 **Systemd Sandboxing Integration** — Automatically configures systemd drop-in override files (`/etc/systemd/system/tomcat.service.d/writable-paths.conf`) to authorize custom writable directories (`ReadWritePaths`) under systemd's strict kernel-level sandboxing
- 🔒 **Enterprise Security** — `no_log` for sensitive files, Vault-ready passwords, root-owned configs with `0640` permissions
- 📦 **JDBC Drivers** — Optional download and installation of JDBC driver JARs to `CATALINA_BASE/lib/`
- 🔄 **Logrotate Support** — Optional per-application log rotation using `/etc/logrotate.d/` with customizable retention, frequency, compression, and olddir
- 🧹 **Purge Mode** — Optionally removes unmanaged context XML files and logrotate configurations (strict mode)
- ✅ **Full Validation** — Comprehensive `assert.yml` validates all inputs before deployment (type, pattern, path traversal protection)
- 🧪 **Molecule Tested** — Automated integration tests with Rocky Linux 9

## 📋 Requirements

- **Ansible**: 2.16 or higher
- **Python**: 3.6 or higher on target hosts (see compatibility note below)
- **Privileges**: sudo/root access on target hosts (for writing to Tomcat directories)
- **Network**: Internet access for downloading JDBC drivers (when configured)

### Supported operating systems

| OS Family                      | Version | Status                                               |
| ------------------------------ | ------- | ---------------------------------------------------- |
| EL (RHEL, Rocky, Alma, Oracle) | 9       | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| EL (RHEL, Rocky, Alma, Oracle) | 8       | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) * |

> **Note**: This role targets EL8 and EL9. It manages application-level configuration within an existing Tomcat instance installed by the `apache_tomcat` role.
>
> \* **EL8 Compatibility Constraints**:
> - **Ansible & Python compatibility**: EL8 defaults to Python 3.6. Support for target Python 3.6 was dropped in `ansible-core` >= 2.17.
>   - To run on EL8 with the system's default Python 3.6, you must use `ansible-core` <= 2.16.
>   - If running `ansible-core` >= 2.17, EL8 targets require Python >= 3.7. However, the system `python3-dnf` package manager bindings on EL8 are compiled exclusively for Python 3.6 and will not be available on newer Python interpreters. This will cause tasks using the `dnf` module (such as installing Java or package updates in other roles or molecule scenarios) to fail.
> - **Molecule Testing**: Due to these Python version compatibility constraints, EL8 is not officially tested in the role's Molecule test suite (which runs a newer Ansible version in CI).

### Ansible version

Ansible >= 2.16

### Python version

Python >= 3.6 (with compatibility constraints on EL8)

### Setup module

The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.

### Root access

This role requires root access for writing context XML files, config files, and JDBC drivers to Tomcat directories. Tasks use `become: true` only where required (Least Privilege Execution).

### Dependencies

- **[grzegorzfranus.apache_tomcat](../ansible-role-apache-tomcat/)** — Must be executed first to install and configure the Tomcat server instance. Declared as a role dependency in `meta/main.yml`.

## 🚀 Quick Start

### 1. Basic Installation

```yaml
---
- name: Deploy Application Contexts
  hosts: tomcat_servers
  roles:
    - role: grzegorzfranus.apache_tomcat_app
      vars:
        tomcat_app_contexts:
          - name: "myapp"
            doc_base: "myapp.war"
```

### 2. Run the playbook

```bash
ansible-playbook -i inventory app-deploy.yml
```

This will deploy a context XML for `myapp` with:

- Context XML in `conf/Catalina/localhost/myapp.xml`
- `reloadable="false"` (security default)
- File permissions `root:tomcat 0640`

## ⚙️ Configuration

### Default Configuration

The role comes with secure, production-ready defaults:

```yaml
# No contexts deployed by default
tomcat_app_contexts: []

# No JDBC drivers by default
tomcat_app_jdbc_drivers: []

# Context XML permissions
tomcat_app_context_file_mode: "0640"

# Config file permissions
tomcat_app_config_file_mode: "0640"

# Do not purge unmanaged contexts
tomcat_app_purge_unmanaged: false
```

### Advanced Configuration — Spring Boot with DataSource

```yaml
---
- name: Deploy Spring Boot Application
  hosts: tomcat_servers
  vars:
    tomcat_app_jdbc_drivers:
      - name: "postgresql"
        url: "https://jdbc.postgresql.org/download/postgresql-42.7.5.jar"
        checksum: "sha256:XXXX"
        filename: "postgresql-42.7.5.jar"

    tomcat_app_contexts:
      - name: "reports"
        doc_base: "reports.war"
        spring_config_location: true

        config_files:
          - filename: "application.properties"
          - filename: "logback-spring.xml"

        datasources:
          - jndi_name: "jdbc/ReportsDB"
            driver_class: "org.postgresql.Driver"
            url: "jdbc:postgresql://192.0.2.10:5432/reports"
            username: "reports_svc"
            password: "{{ vault_reports_db_password }}"
            max_total: 50
            max_idle: 10
            min_idle: 5

        environments:
          - name: "config/appMode"
            type: "java.lang.String"
            value: "production"

        parameters:
          - name: "env"
            value: "production"
            override: false

  roles:
    - role: grzegorzfranus.apache_tomcat_app
```

### Minimal Configuration — Basic Context Only

```yaml
---
- name: Deploy basic context
  hosts: tomcat_servers
  vars:
    tomcat_app_contexts:
      - name: "myapp"
        doc_base: "myapp.war"
  roles:
    - role: grzegorzfranus.apache_tomcat_app
```

## 📊 Variables

### Tomcat Integration

| Variable                   | Description                                                             | Default               |
| -------------------------- | ----------------------------------------------------------------------- | --------------------- |
| `tomcat_app_catalina_base` | Full path to CATALINA_BASE (must match the `apache_tomcat` role value)  | `"/opt/tomcat/instance"` |
| `tomcat_app_user`          | System user that runs the Tomcat process                                | `"tomcat"`            |
| `tomcat_app_group`         | System group for the Tomcat process                                     | `"tomcat"`            |
| `tomcat_app_service_name`  | Name of the systemd service unit                                        | `"tomcat"`            |

### Application Contexts

| Variable              | Description                                                      | Default |
| --------------------- | ---------------------------------------------------------------- | ------- |
| `tomcat_app_contexts` | List of application context definitions (see sub-tables below)   | `[]`    |

#### Context Properties

Each element in `tomcat_app_contexts` supports the following properties:

| Property                 | Description                                                              | Required | Default  |
| ------------------------ | ------------------------------------------------------------------------ | -------- | -------- |
| `name`                   | Name of the context XML file (without `.xml` extension)                  | ✅ Yes   | —        |
| `doc_base`               | `docBase` attribute for the Context element (e.g. `myapp.war`)           | No       | `""`     |
| `reload_on_change`       | Set `reloadable` attribute. `false` recommended for security             | No       | `false`  |
| `privileged`             | Whether this context runs as a privileged context                        | No       | `false`  |
| `override`               | Whether to override Host-level configuration                             | No       | `true`   |
| `anti_resource_locking`  | Enable `antiResourceLocking` for hot-redeploy support                    | No       | `false`  |
| `spring_config_location` | Auto-add `spring.config.additional-location` pointing to `config/<name>/` | No      | `false`  |
| `writable_dirs`          | List of custom writable directory definitions (see sub-table)            | No       | `[]`     |
| `config_files`           | List of config file definitions (see sub-table)                          | No       | `[]`     |
| `datasources`            | List of JNDI DataSource definitions (see sub-table)                      | No       | `[]`     |
| `environments`           | List of JNDI Environment entry definitions (see sub-table)               | No       | `[]`     |
| `resource_links`         | List of Resource Link definitions (see sub-table)                        | No       | `[]`     |
| `parameters`             | List of Context Parameter definitions (see sub-table)                    | No       | `[]`     |
| `logrotate`              | Optional logrotate configuration definition (see sub-table)             | No       | —        |

#### Logrotate Properties

Each element in `logrotate` supports the following properties:

| Property | Description | Required | Default |
|---|---|---|---|
| `log_paths` | List of log file glob patterns to rotate (e.g. `['/var/log/myapp/*.log']`). | Yes | — |
| `frequency` | Rotation frequency: `hourly`, `daily`, `weekly`, or `monthly`. | No | `"daily"` |
| `count` | Number of rotated log files to keep. | No | `14` |
| `compress` | Compress rotated log files with gzip. | No | `true` |
| `missingok` | Do not issue an error if the log file is missing. | No | `true` |
| `copytruncate` | Truncate the original log file after creating a copy. | No | `true` |
| `archive_directory_path` | Absolute path to move rotated files to (`olddir`). | No | `""` |
| `su` | Enable su option in logrotate config. | No | `true` |

#### Writable Directories Properties

The role ensures that these directories exist with the correct owner/group, and configures a systemd drop-in override file (`/etc/systemd/system/tomcat.service.d/writable-paths.conf`) that appends them to systemd's `ReadWritePaths` directive to allow write access under strict systemd sandboxing. Note: Paths under `/tmp` or `/var/tmp` are automatically skipped, as they are already writable by the service due to `PrivateTmp=true` and would cause namespace loading errors.

| Property | Description                                                             | Required | Default |
| -------- | ----------------------------------------------------------------------- | -------- | ------- |
| `path`   | Absolute path to the custom writable directory                          | ✅ Yes   | —       |
| `owner`  | Owner of the directory                                                  | No       | `tomcat` (from `tomcat_app_user`) |
| `group`  | Group of the directory                                                  | No       | `tomcat` (from `tomcat_app_group`) |
| `mode`   | Octal permissions for the directory                                     | No       | `"0750"` |

#### Config Files Properties

The role creates empty files with correct ownership (`root:tomcat`) and permissions if they do not exist. **File content is NOT managed by this role** — it is the user's or CI/CD pipeline's responsibility.

| Property   | Description                                                           | Required | Default |
| ---------- | --------------------------------------------------------------------- | -------- | ------- |
| `filename` | Filename to ensure exists in the per-app config directory (`config/<name>/`) | ✅ Yes   | —       |

#### DataSource Properties

Each element in `datasources` supports:

| Property                   | Description                                                | Required | Default                                         |
| -------------------------- | ---------------------------------------------------------- | -------- | ----------------------------------------------- |
| `jndi_name`                | JNDI name for the DataSource (e.g. `jdbc/MyDB`)           | ✅ Yes   | —                                               |
| `driver_class`             | Fully qualified JDBC driver class name                     | ✅ Yes   | —                                               |
| `url`                      | JDBC connection URL                                        | ✅ Yes   | —                                               |
| `username`                 | Database username                                          | ✅ Yes   | —                                               |
| `password`                 | Database password (use Ansible Vault!)                     | ✅ Yes   | —                                               |
| `max_total`                | Maximum number of active connections in the pool           | No       | `50`                                            |
| `max_idle`                 | Maximum number of idle connections                         | No       | `10`                                            |
| `min_idle`                 | Minimum number of idle connections                         | No       | `5`                                             |
| `max_wait_millis`          | Maximum wait time in ms for a connection                   | No       | `10000`                                         |
| `validation_query`         | SQL query used to validate connections                     | No       | `"SELECT 1"`                                    |
| `validation_interval`      | Interval in ms between validation checks                   | No       | `30000`                                         |
| `test_on_borrow`           | Validate connections when borrowed from the pool           | No       | `true`                                          |
| `test_while_idle`          | Validate idle connections periodically                     | No       | `true`                                          |
| `remove_abandoned`         | Remove abandoned (leaked) connections                      | No       | `true`                                          |
| `remove_abandoned_timeout` | Timeout in seconds before a connection is considered abandoned | No   | `300`                                           |
| `log_abandoned`            | Log stack trace of code that abandoned a connection        | No       | `true`                                          |

#### Environment Entry Properties

Each element in `environments` supports:

| Property   | Description                                                     | Required | Default |
| ---------- | --------------------------------------------------------------- | -------- | ------- |
| `name`     | JNDI name of the environment entry                              | ✅ Yes   | —       |
| `type`     | Fully qualified Java type (e.g. `java.lang.String`)             | ✅ Yes   | —       |
| `value`    | Value of the environment entry                                  | ✅ Yes   | —       |
| `override` | Whether the application can override this value                 | No       | `false` |

#### Resource Link Properties

Each element in `resource_links` supports:

| Property | Description                                     | Required | Default |
| -------- | ----------------------------------------------- | -------- | ------- |
| `name`   | JNDI name of the resource in this context       | ✅ Yes   | —       |
| `global` | Global JNDI name of the linked resource         | ✅ Yes   | —       |
| `type`   | Fully qualified Java type of the resource       | ✅ Yes   | —       |

#### Context Parameter Properties

Each element in `parameters` supports:

| Property   | Description                                         | Required | Default |
| ---------- | --------------------------------------------------- | -------- | ------- |
| `name`     | Parameter name                                      | ✅ Yes   | —       |
| `value`    | Parameter value                                     | ✅ Yes   | —       |
| `override` | Whether the application can override this parameter | No       | `false` |

### JDBC Drivers

| Variable                  | Description                                                          | Default |
| ------------------------- | -------------------------------------------------------------------- | ------- |
| `tomcat_app_jdbc_drivers` | Optional list of JDBC driver JARs to install in `CATALINA_BASE/lib/` | `[]`    |

Each element in `tomcat_app_jdbc_drivers` supports:

| Property   | Description                                                    | Required | Default |
| ---------- | -------------------------------------------------------------- | -------- | ------- |
| `name`     | Human-readable identifier for this driver                      | ✅ Yes   | —       |
| `url`      | Download URL for the JDBC driver JAR                           | ✅ Yes   | —       |
| `filename` | Destination filename in `CATALINA_BASE/lib/`                   | ✅ Yes   | —       |
| `checksum` | Checksum for download verification (format: `sha256:HASH`)     | No       | —       |

### Context Management

| Variable                       | Description                                                              | Default  |
| ------------------------------ | ------------------------------------------------------------------------ | -------- |
| `tomcat_app_context_file_mode` | File permissions for context XML files in `conf/Catalina/localhost/`      | `"0640"` |
| `tomcat_app_config_file_mode`  | File permissions for application config files (`application.properties`) | `"0640"` |
| `tomcat_app_purge_unmanaged`   | Remove context XML files not defined in `tomcat_app_contexts`            | `false`  |

## 📌 Role Properties

| Property         | Value        | Description                                                                                          |
| ---------------- | ------------ | ---------------------------------------------------------------------------------------------------- |
| **Idempotent**   | ✅ Yes       | Re-running produces no changes if configuration is unchanged                                         |
| **Atomic**       | ✅ Yes       | Failures leave system in a consistent state (template + file operations)                             |
| **Check Mode**   | ✅ Supported | All template tasks work in check mode                                                                |
| **Diff Mode**    | ✅ Supported | Template tasks support diff mode for change preview                                                  |

## 📤 Role Output

This role does not set any public output facts. All internal facts use the `__tomcat_app_` double-underscore prefix and are not part of the public interface.

## 🔍 Verification

After deployment, verify application contexts are working correctly:

### Check Context XML

```bash
# View deployed context XML
cat /opt/tomcat/instance/conf/Catalina/localhost/myapp.xml

# Check permissions (should be 0640 root:tomcat)
ls -la /opt/tomcat/instance/conf/Catalina/localhost/myapp.xml
```

### Check Config Files

```bash
# View per-app config directory
ls -la /opt/tomcat/instance/config/myapp/

# Verify application.properties was deployed
cat /opt/tomcat/instance/config/myapp/application.properties
```

### Check JDBC Drivers

```bash
# Verify driver JARs in lib directory
ls -la /opt/tomcat/instance/lib/*.jar
```

### Verify Tomcat Loaded the Context

```bash
# Check Tomcat logs for context loading
journalctl -u tomcat --no-pager -n 50 | grep myapp
```

## 🛡️ Security Features

- ✅ **Passwords protected**: `no_log: true` on context XML deployment (DataSource credentials) and sensitive config files
- ✅ **Ansible Vault ready**: All password fields (`password` in DataSources) designed for Vault override
- ✅ **Least Privilege Execution**: `become: true` only on tasks that write to system directories (6 tasks). Assert, find, set_fact — without become
- ✅ **Root-owned configs**: Context XMLs and config files owned by `root:tomcat` with `0640` permissions
- ✅ **Path traversal protection**: `assert.yml` blocks `../` in config file filenames
- ✅ **Reloadable disabled by default**: `reload_on_change` defaults to `false` (CIS hardening)
- ✅ **Config file isolation**: Each application gets its own directory (`config/<name>/`) — no classpath cross-contamination
- ✅ **Input validation**: Comprehensive `assert.yml` validates all variable types, patterns, and required fields before any changes

### Uninstall

This role does not include a built-in uninstall mechanism. To remove application contexts:

```bash
# Remove specific context XML
sudo rm /opt/tomcat/instance/conf/Catalina/localhost/myapp.xml

# Remove per-app config directory
sudo rm -rf /opt/tomcat/instance/config/myapp/

# Remove JDBC drivers (if installed by this role)
sudo rm /opt/tomcat/instance/lib/postgresql-*.jar

# Restart Tomcat to apply changes
sudo systemctl restart tomcat
```

Alternatively, enable purge mode to let the role remove all unmanaged contexts and logrotate configurations:

```yaml
tomcat_app_purge_unmanaged: true
tomcat_app_contexts: [] # empty = remove all contexts and logrotate configurations
```

### Roll-back Capabilities

This role **does not** support automatic rollback. Context XML files are re-rendered from templates on each run. To revert to a previous configuration:

1. Restore your previous variable values in your inventory/playbook
2. Re-run the role — it will overwrite context XMLs and config files with the previous configuration

## CI/CD Pipeline

This repository uses centralized, reusable GitHub Actions workflows from [grzegorzfranus/github-workflows](https://github.com/grzegorzfranus/github-workflows) (`v3.0.1`) for quality assurance, security scanning, and release automation.

### CI Pipeline (`ansible-ci.yml@v3.0.1`)

Runs on every Pull Request in a two-tier gate pattern:

1. **Branch Name Lint** — enforces naming conventions (`feature/`, `bugfix/`, `fix/`, `hotfix/`, `release/`, `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`, `revert/`)
2. **PR Title Lint** — enforces [Conventional Commits](https://www.conventionalcommits.org/) format (`feat:`, `fix:`, `ci:`, etc.)
3. **YAML Syntax Lint** — validates YAML formatting via `yamllint`
4. **Ansible Lint** — checks Ansible best practices and role standards
5. **Galaxy Metadata Validation** — verifies `meta/main.yml` schema and requirements (`ansible-meta-validate.yml`)
6. **Security Scanning** — TruffleHog secret detection and Trivy IaC scanning (`ansible-security.yml`)
7. **Molecule Integration Tests** — executes Molecule test matrix across Rocky Linux 9 (`ansible-molecule.yml`)
8. **Merge Check Gate** — single authoritative status check aggregating all results for branch protection

### Release & Publish Pipeline (`ansible-publish.yml@v3.0.1`)

Automated via [Release Please](https://github.com/googleapis/release-please):

1. **Push to `main`** → Release Please creates or updates a Release PR with automated changelog generation
2. **Release PR Validation** → validates YAML syntax and actions schema before setting `Merge Check` status
3. **Merge Release PR** → creates Git version tag and GitHub Release automatically
4. **Ansible Galaxy Publish** → publishes tagged release to Ansible Galaxy via `ansible-publish.yml@v3.0.1` with exponential backoff retry logic

## Example Playbooks

### Basic Context Only

```yaml
---
- name: Deploy basic application context
  hosts: tomcat_servers
  roles:
    - role: grzegorzfranus.apache_tomcat_app
      vars:
        tomcat_app_contexts:
          - name: "myapp"
            doc_base: "myapp.war"
```

### Spring Boot with Externalized Config

```yaml
---
- name: Deploy Spring Boot app with externalized properties
  hosts: tomcat_servers
  vars:
    tomcat_app_contexts:
      - name: "myapp"
        doc_base: "myapp.war"
        spring_config_location: true
        config_files:
          - filename: "application.properties"
          - filename: "logback-spring.xml"
  roles:
    - role: grzegorzfranus.apache_tomcat_app
```

### Spring Boot with Logrotate Config

```yaml
---
- name: Deploy Spring Boot app with logrotate
  hosts: tomcat_servers
  vars:
    tomcat_app_contexts:
      - name: "myapp"
        doc_base: "myapp.war"
        writable_dirs:
          - path: "/var/log/myapp"
        logrotate:
          log_paths:
            - "/var/log/myapp/*.log"
          frequency: "daily"
          count: 14
          compress: true
          missingok: true
          copytruncate: true
          archive_directory_path: "/var/log/myapp/archive"
  roles:
    - role: grzegorzfranus.apache_tomcat_app
```

### Multi-Application with JDBC Drivers

```yaml
---
- name: Deploy multiple applications
  hosts: tomcat_servers
  vars:
    tomcat_app_purge_unmanaged: true

    tomcat_app_jdbc_drivers:
      - name: "postgresql"
        url: "https://jdbc.postgresql.org/download/postgresql-42.7.5.jar"
        checksum: "sha256:abc123..."
        filename: "postgresql-42.7.5.jar"

    tomcat_app_contexts:
      - name: "reports"
        doc_base: "reports.war"
        spring_config_location: true
        config_files:
          - filename: "application.properties"
        datasources:
          - jndi_name: "jdbc/ReportsDB"
            driver_class: "org.postgresql.Driver"
            url: "jdbc:postgresql://192.0.2.10:5432/reports"
            username: "reports_svc"
            password: "{{ vault_reports_db_password }}"
        environments:
          - name: "config/appMode"
            type: "java.lang.String"
            value: "production"

      - name: "api"
        doc_base: "api.war"
        datasources:
          - jndi_name: "jdbc/ApiDB"
            driver_class: "org.postgresql.Driver"
            url: "jdbc:postgresql://192.0.2.10:5432/api"
            username: "api_svc"
            password: "{{ vault_api_db_password }}"

  roles:
    - role: grzegorzfranus.apache_tomcat_app
```

## 📁 File Structure

```
ansible-role-apache-tomcat-app/
├── .github/
│   ├── ISSUE_TEMPLATE/                # Issue templates for bug, feature, task
│   │   ├── bug_report.yml
│   │   ├── config.yml
│   │   ├── feature_request.yml
│   │   └── task.yml
│   ├── PULL_REQUEST_TEMPLATE/         # Pull request description template
│   │   └── pull_request_template.md
│   ├── workflows/
│   │   ├── ci.yml                     # CI pipeline
│   │   └── release.yml                # Release Please + Galaxy publish
│   └── dependabot.yml                 # Dependabot configuration for GitHub Actions
├── .ansible-lint                      # Ansible lint configuration
├── .gitignore                         # Git ignore rules
├── .release-please-manifest.json      # Release Please version manifest
├── .yamllint                          # YAML lint configuration
├── CHANGELOG.md                       # Version history
├── LICENSE                            # Apache-2.0 license
├── README.md                          # This documentation
├── release-please-config.json         # Release Please configuration
├── defaults/
│   └── main.yml                       # Default variables (4 sections)
├── handlers/
│   └── main.yml                       # Tomcat service restart
├── meta/
│   ├── main.yml                       # Galaxy metadata + dependencies
│   └── argument_specs.yml             # Full variable specification
├── molecule/
│   └── default/                       # Default test scenario
│       ├── molecule.yml
│       ├── converge.yml
│       ├── prepare.yml
│       ├── verify.yml
│       └── templates/
│           └── testapp/               # Test Jinja2 templates
│               ├── application.properties.j2
│               └── logback-spring.xml.j2
├── tasks/
│   ├── main.yml                       # Task orchestration (assert → jdbc → config → contexts → purge)
│   ├── assert.yml                     # Variable validation (all defaults verified)
│   ├── jdbc_drivers.yml               # JDBC driver download and installation
│   ├── config_files.yml               # Per-app config directory management
│   ├── config_files_deploy.yml        # Inner loop: template config files
│   ├── contexts.yml                   # Context XML deployment
│   └── purge.yml                      # Unmanaged context cleanup
├── templates/
│   └── context.xml.j2                 # Context XML template
└── vars/
    └── main.yml                       # Internal variables (__tomcat_app_ prefix)
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
  - `feat:` — new features (minor version bump)
  - `fix:` — bug fixes (patch version bump)
  - `docs:` — documentation changes
  - `refactor:` — code refactoring
  - `test:` — test additions
  - `ci:` — CI/CD changes
  - `chore:` — maintenance tasks
- Use branch naming convention: `feature/`, `bugfix/`, `hotfix/`, `docs/`, `refactor/`, `test/`, `chore/`, `ci/`
- Ensure your code passes all CI checks (YAML lint, Ansible lint, Molecule tests)
- Centralized workflows from [github-workflows](https://github.com/grzegorzfranus/github-workflows) version `v3.0.1` are used to run CI/CD pipelines
- Submit a pull request describing your changes (a template is available under `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md` to help structure your PR description)
- For major changes, please open an issue first to discuss what you would like to change (issue templates for bug reports, feature requests, and tasks are available under `.github/ISSUE_TEMPLATE/`)

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
