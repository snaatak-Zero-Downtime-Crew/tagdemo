# SonarQube Setup Role

This Ansible role installs and configures **SonarQube** on a target machine. SonarQube is a popular open-source platform used for continuous inspection of code quality. It performs automatic reviews of code to detect bugs, vulnerabilities, and code smells in your codebase.

## Requirements

- This role requires **Java** (at least version 11) to be installed on the target machine, as SonarQube is built on Java.
- If you are setting up SonarQube to run as a service, you may need to configure additional system settings (e.g., firewall, system user) which are outside the scope of this role.

## Role Variables

The following variables can be defined for customizing your SonarQube installation:

### `sonarqube_version`
- Default: `latest`
- The version of SonarQube to install. You can specify a specific version or use `latest` to always get the newest stable release.

### `sonarqube_install_dir`
- Default: `/opt/sonarqube`
- The directory where SonarQube will be installed.

### `sonarqube_user`
- Default: `sonar`
- The system user that will be used to run the SonarQube service.

### `sonarqube_group`
- Default: `sonar`
- The system group to assign ownership to.

### `sonarqube_service_enabled`
- Default: `true`
- Whether to configure SonarQube as a system service to start automatically on boot.

### `sonarqube_db_url`
- Default: `jdbc:postgresql://localhost:5432/sonar`
- The JDBC URL for the PostgreSQL database used by SonarQube. You can change this to suit your database configuration.

### `sonarqube_db_username`
- Default: `sonar`
- The username for connecting to the SonarQube database.

### `sonarqube_db_password`
- Default: `sonar`
- The password for the SonarQube database user.

## Dependencies

`postgresql`: If you are using PostgreSQL as the database for SonarQube, the postgresql role should be installed.

`java`: You need a Java role or task to install OpenJDK 11 or later on the target machine.

## Example Playbook

Here is an example playbook that installs and configures SonarQube:

```yaml
- hosts: all
  become: true
  vars:
    sonarqube_version: "9.9.0.65466"  # Specify your preferred version
    sonarqube_db_url: "jdbc:postgresql://localhost:5432/sonar"
    sonarqube_db_username: "sonar"
    sonarqube_db_password: "sonar"

  roles:
    - { role: sonarqube, sonarqube_service_enabled: true }
```

This will install SonarQube, configure it with the specified database settings, and set it up as a service.


## Author Information

This role was created by **[Aayush verma ]**. For any inquiries or issues, please reach out at **[aayush.verma@mygurukulam.co]**.

---

This template includes a basic setup for SonarQube using Ansible. You can expand on it based on your environment and requirements (e.g., different database types or additional configuration).
