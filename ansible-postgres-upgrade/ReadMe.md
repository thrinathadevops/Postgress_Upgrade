PostgreSQL Upgrade Role
-----------------------
This Ansible role automates the process of upgrading PostgreSQL from an older version to a newer one. The process includes pre-checks, backup, upgrade, and post-upgrade validation.

Directory Structure
-------------------


ansible-postgres-upgrade/

├── ReadMe.md                                  # Documentation about the structure, usage, and steps

├── playbooks/

│   ├── upgrade_main.yml                       # Main playbook to execute the upgrade

│   ├── validate_prechecks.yml                 # Playbook for environment prechecks

│   ├── post_upgrade_validation.yml            # Playbook for post-upgrade validations

├── roles/

│   ├── defaults/

│   │   └── main.yml                           # Default variables for the role

│   ├── files/

│   │   ├── postgresql.conf.j2                 # Template for postgresql.conf

│   │   ├── postgresql-13_orig.conf            # Original configuration backup

│   ├── handlers/

│   │   └── main.yml                           # Handlers for restarting services or notifications

│   ├── logs/

│   │   ├── upgrade.log                        # Captures detailed task output

│   │   ├── precheck.log                       # Logs precheck outputs

│   │   ├── validation.log                     # Logs validation steps

│   ├── tasks/

│   │   ├── backup_postgres_extensions.yml     # Task for extension backups

│   │   ├── pre_upgrade_tasks.yml              # Tasks to prepare the environment for the upgrade

│   │   ├── post_upgrade_tasks.yml             # Tasks for validations and final setup

│   │   ├── check_postgres_version.yml         # Task to check PostgreSQL version

│   │   ├── init_new_postgres_db.yml           # Initialize and validate the new PostgreSQL database

│   │   ├── pg_upgrade.yml                     # Task for the actual `pg_upgrade`

│   │   ├── start_stop_postgres.yml            # Unified task to start/stop PostgreSQL

│   │   ├── validate_upgrade.yml               # Validate upgrade steps

│   │   ├── manage_storage.yml                 # Verify and manage storage

│   │   ├── monitor_sessions.yml               # Fetch and log active sessions

│   │   ├── vacuum_analyze.yml                 # Perform vacuum and analyze

│   │   └── main.yml                           # Entry point for all tasks

│   ├── templates/

│   │   └── postgresql.conf.j2                 # Jinja2 template for configuration

│   ├── vars/

│   │   └── main.yml                           # Variables specific to the role

│   └── reports/

│       ├── precheck_report.md                 # Markdown file summarizing prechecks

│       ├── upgrade_summary.md                 # Markdown summary of the upgrade process

│       ├── validation_report.md               # Report post-validation

└── inventory/

    ├── hosts                                  # Inventory file defining servers
    
    ├── group_vars/
    
    │   └── all.yml                            # Global variables
    
    └── host_vars/
    
        └── specific_host.yml                  # Host-specific variables

Pre-requisites
--------------
1. Ansible should be installed on your control machine.
2. Ensure that the PostgreSQL servers are accessible and have SSH enabled.
3. Ensure that the user has necessary permissions to manage PostgreSQL and perform system-level tasks.

How to Use
----------
Inventory Setup
---------------
Define your PostgreSQL servers in the inventory/hosts file. Here is an example:
1. [postgres_servers]
2. server1.example.com
3. server2.example.com

Configuration Setup
-------------------
Set any necessary host or global variables in the inventory/group_vars/all.yml or inventory/host_vars/specific_host.yml files. You may need to modify the following variables:
  1. postgres_old_version: The version of PostgreSQL you are upgrading from.
  2. postgres_new_version: The version of PostgreSQL you are upgrading to.
  3. postgres_data_dir: The data directory of PostgreSQL.
  
Precheck Execution
------------------
  Before proceeding with the actual upgrade, run the precheck playbook to validate the environment
          ansible-playbook playbooks/validate_prechecks.yml -i inventory/hosts
This playbook performs the following tasks:
  1. Confirms that the PostgreSQL binaries for the target version are available.
  2. Checks disk space requirements in the postgres_data_dir and /tmp directories.
  3. Verifies that no active connections exist on the current PostgreSQL instance.
  4. Ensures required libraries and extensions are compatible with the new version.
Precheck Log Location
  roles/logs/precheck.log

Playbook Execution
------------------
ansible-playbook playbooks/upgrade_main.yml -i inventory/hosts
This will:
    1. Perform environment prechecks.
    2. Backup the necessary extensions and configurations.
    3. Execute the PostgreSQL upgrade.
    4. Validate the upgrade process.
    
Post-upgrade Validation
-----------------------
After the upgrade, the post-upgrade validation playbook post_upgrade_validation.yml can be run to ensure the upgrade was successful:
ansible-playbook playbooks/post_upgrade_validation.yml -i inventory/hosts

Logs and Reports
----------------
The upgrade process logs will be available in the roles/logs/ directory. Reports summarizing the prechecks, upgrade summary, and post-validation steps are available in the roles/reports/ directory.
  1. upgrade.log: Captures detailed task outputs.
  2. precheck.log: Logs precheck outputs.
  3. validation.log: Logs post-validation steps.
  
Troubleshooting
---------------
  1. Ensure that PostgreSQL is stopped before starting the upgrade process.
  2. If the upgrade fails, check the logs in roles/logs/upgrade.log for detailed error messages.
  3. Review the roles/reports/upgrade_summary.md for a summary of the upgrade steps.
  
