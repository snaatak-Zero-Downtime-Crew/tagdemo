# Documentation on SonarQube Setup using Ansible role


| **Author** | **Created on** | **Version** | **Last updated by**|**Last Edited On**|**Level** |**Reviewer** |
|------------|---------------------------|-------------|----------------|-----|-------------|-------------|
| Aayush Verma|   15-03-2025              | v1          | Aayush Verma   | 15-03-2025   |  Internal Reviewer | Siddharth |


## **Table of Contents**  

1. [Introduction](#introduction)  
2. [Pre-requisites](#pre-requisites)  
3. [System Requirements](#system-requirements)  
4. [Dependencies](#dependencies)  
5. [Runtime Dependencies](#runtime-dependencies)  
6. [Important Ports](#important-ports)
7. [SonarQube](#sonarqube)  
   - [Key Features of SonarQube](#key-features-of-sonarqube)  
   - [Important Configuration Files in SonarQube](#important-configuration-files-in-sonarqube)
8. [Install SonarQube](#install-sonarqube)
9. [Install Ansible](#install-ansible)
10. [Flow Diagram](#flow-diagram) 
11. [Creating a Role](#creating-a-role)  
12. [Folder Structure](#folder-structure)  
13. [Variables](#variables)
14. [Subtask Files](#subtasks-files)
15. [Templates for Configuration](#templates-for-configuration)  
16. [Run Playbook](#run-playbook)  
17. [Conclusion](#conclusion)  
18. [Contact Information](#contact-information)  
19. [References](#references)  



## Introduction
This document guides the automated deployment of SonarQube using Ansible, covering installation, configuration, and service setup. The role ensures efficient management across environments with minimal manual effort.



## Pre-requisites
Before setting up SonarQube using Ansible, ensure you have the following:

- Ansible installed on your local machine
- A target Ubuntu server (20.04 or later) with a non-root user having sudo privileges




## System Requirements
The system requirements depend on the size of the codebase being analyzed. The minimum and recommended configurations are:

| **Component**  | **Minimum Requirement** | **Recommended** |
|---------------|------------------------|----------------|
| **CPU**       | 2 vCPUs                 | 4 vCPUs or more |
| **RAM**       | 4 GB                     | 8 GB or more   |
| **Storage**   | 10 GB free disk space   | 50 GB or more  |
| **OS**        | Ubuntu 20.04 | Latest LTS version |



## Dependencies
Ensure the following dependencies are installed before running the Ansible role:

- Python (v3.6+)
- Ansible (v2.9+)



## Runtime Dependencies
Once SonarQube is running, the following services should be available:

- SonarQube Server
- OpenJDK (v11+)
- PostgreSQL Database
- Reverse Proxy (if applicable)



## Important Ports
To ensure proper connectivity, the following ports need to be opened:

| **Port** | **Service** | **Description** |
|----------|------------|----------------|
| **9000** | SonarQube | Default web UI and API access |
| **5432** | PostgreSQL | Database communication |
| **80/443** | Nginx (Optional) | If using a reverse proxy |






## SonarQube 
SonarQube is a leading open-source platform for continuous inspection of code quality. It analyzes codebases, identifies bugs, security vulnerabilities, and code smells. Offering a comprehensive view of code health, SonarQube assists development teams in maintaining high-quality software, ensuring robust security, and fostering continuous improvement in codebases.

## Key Features of SonarQube:

1. **Code Quality Analysis**: Detects bugs, vulnerabilities, and code smells.
2. **Multi-Language Support**: Supports over 20 programming languages.
3. **Integration with CI/CD**: Can be integrated with Jenkins, GitLab CI, Azure DevOps, etc.
4. **Custom Rules**: Allows you to define custom rules for code analysis.
5. **Dashboards and Reports**: Provides detailed dashboards and reports for code quality metrics.


## Important Configuration Files in SonarQube

1. **`sonar-project.properties`**:
   This is the main configuration file for a SonarQube project. It defines the project settings, such as the project key, name, version, and source directories.

   ```properties
   # Required metadata
   sonar.projectKey=my_project_key
   sonar.projectName=My Project
   sonar.projectVersion=1.0

   # Path to source directories (comma-separated)
   sonar.sources=src

   # Path to test directories (comma-separated)
   sonar.tests=test

   # Encoding of the source files
   sonar.sourceEncoding=UTF-8

   # Language of the project
   sonar.language=java

   # Exclude specific files or directories
   sonar.exclusions=**/test/**, **/generated/**

   # Include specific files or directories
   sonar.inclusions=**/*.java, **/*.js

   # Additional properties for specific languages or plugins
   sonar.java.binaries=target/classes
   sonar.java.libraries=target/lib/*.jar
   ```


| **Property**                  | **Use Case**                                                                                     | **Example Value**                          | **Description**                                                                 |
|-------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------|
| `sonar.projectKey`             | Unique identifier for the project in SonarQube.                                                 | `my_project_key`                           | Used to uniquely identify the project in SonarQube.                             |
| `sonar.projectName`            | Display name for the project in SonarQube.                                                      | `My Project`                               | The name of the project as it appears in the SonarQube UI.                      |
| `sonar.projectVersion`         | Version of the project being analyzed.                                                          | `1.0`                                      | Helps track different versions of the project in SonarQube.                     |
| `sonar.sources`                | Specifies the directories containing source code.                                               | `src`                                      | Tells SonarQube where to find the source code for analysis.                     |
| `sonar.tests`                  | Specifies the directories containing test code.                                                 | `test`                                     | Tells SonarQube where to find the test code for analysis.                       |
| `sonar.sourceEncoding`         | Specifies the encoding of the source files.                                                     | `UTF-8`                                    | Ensures SonarQube reads files with the correct encoding.                        |
| `sonar.language`               | Specifies the primary language of the project.                                                  | `java`                                     | Helps SonarQube apply the correct rules and analyzers for the language.         |
| `sonar.exclusions`             | Excludes specific files or directories from analysis.                                           | `**/test/**, **/generated/**`              | Prevents SonarQube from analyzing test or generated code.                       |
| `sonar.inclusions`             | Includes specific files or directories for analysis.                                            | `**/*.java, **/*.js`                       | Limits analysis to specific file types or directories.                          |
| `sonar.java.binaries`          | Specifies the directory containing compiled Java classes.                                       | `target/classes`                           | Required for Java projects to analyze compiled code.                            |
| `sonar.java.libraries`         | Specifies the directory containing Java libraries (JAR files).                                  | `target/lib/*.jar`                         | Required for Java projects to analyze dependencies.                            |



2. **`sonar-scanner.properties`**:
   This file is used to configure the SonarQube scanner, which is the tool that performs the code analysis.

   ```properties
   # SonarQube server URL
   sonar.host.url=http://localhost:9000

   # SonarQube server authentication token
   sonar.login=your_token_here

   # Project key (can be overridden in sonar-project.properties)
   sonar.projectKey=my_project_key

   # Project name (can be overridden in sonar-project.properties)
   sonar.projectName=My Project

   # Project version (can be overridden in sonar-project.properties)
   sonar.projectVersion=1.0
   ```
| **Property**                  | **Use Case**                                                                                     | **Example Value**                          | **Description**                                                                 |
|-------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------|
| `sonar.host.url`               | Specifies the URL of the SonarQube server.                                                      | `http://localhost:9000`                    | Defines where the SonarQube server is hosted.                                   |
| `sonar.login`                  | Authentication token for accessing the SonarQube server.                                        | `your_token_here`                          | Used for authenticating with the SonarQube server (e.g., for CI/CD integration).|
| `sonar.projectKey`             | Unique identifier for the project in SonarQube.                                                 | `my_project_key`                           | Used to uniquely identify the project in SonarQube.                             |
| `sonar.projectName`            | Display name for the project in SonarQube.                                                      | `My Project`                               | The name of the project as it appears in the SonarQube UI.                      |
| `sonar.projectVersion`         | Version of the project being analyzed.                                                          | `1.0`                                      | Helps track different versions of the project in SonarQube.                     |



3. **`sonar-qube/conf/sonar.properties`**:
   This file is used to configure the SonarQube server itself. It is typically located in the `conf` directory of your SonarQube installation.

   ```properties
   # Database configuration
   sonar.jdbc.username=sonarqube
   sonar.jdbc.password=your_password
   sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

   # Web server configuration
   sonar.web.host=0.0.0.0
   sonar.web.port=9000

   # Elasticsearch configuration
   sonar.search.host=localhost
   sonar.search.port=9001

   # Security configuration
   sonar.forceAuthentication=true
   sonar.security.realm=LDAP
   sonar.security.localUsers=admin
   ```

| **Property**                  | **Use Case**                                                                                     | **Example Value**                          | **Description**                                                                 |
|-------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------|
| `sonar.jdbc.username`          | Database connection username for SonarQube.                                                    | `sonarqube`                                | Specifies the username for connecting to the database (e.g., PostgreSQL).       |
| `sonar.jdbc.password`          | Database connection password for SonarQube.                                                    | `your_password`                            | Specifies the password for connecting to the database.                          |
| `sonar.jdbc.url`               | Database connection URL for SonarQube.                                                         | `jdbc:postgresql://localhost/sonarqube`    | Specifies the JDBC URL for connecting to the database (e.g., PostgreSQL).       |
| `sonar.web.host`               | Host address for the SonarQube web server.                                                     | `0.0.0.0`                                  | Binds the SonarQube web server to all available IP addresses.                   |
| `sonar.web.port`               | Port for the SonarQube web server.                                                             | `9000`                                     | Specifies the port on which the SonarQube web server will run.                  |
| `sonar.search.host`            | Host address for the Elasticsearch server used by SonarQube.                                   | `localhost`                                | Specifies the host for Elasticsearch (used for search and indexing).            |
| `sonar.search.port`            | Port for the Elasticsearch server used by SonarQube.                                           | `9001`                                     | Specifies the port for Elasticsearch.                                           |
| `sonar.forceAuthentication`    | Enforces authentication for accessing SonarQube.                                               | `true`                                     | Ensures users must log in to access SonarQube.                                  |
| `sonar.security.realm`         | Specifies the authentication realm (e.g., LDAP, local).                                        | `LDAP`                                     | Configures SonarQube to use LDAP for user authentication.                       |
| `sonar.security.localUsers`    | Specifies local users who can log in to SonarQube.                                             | `admin`                                    | Defines local users (e.g., `admin`) for authentication.                         |



## Install SonarQube

 To install SonarQube on your system, please follow the link below for the SonarQube Installation Guide. :- [SonarQube Installation Guide]()


## Install Ansible

 To install Ansible on your system, please follow the link below for the Ansible Installation Guide. :- [Ansible Installation Guide](https://github.com/snaatak-Zero-Downtime-Crew/Documentation/blob/Aayush-SCRUM-93/Common/Software/Ansible/Installation/README.md)


## Flow Diagram

* This diagram should help you visualize the sequence of tasks in the Ansible role for setting up SonarQube.

![image](https://github.com/user-attachments/assets/c591566e-eb68-4044-b7e3-326043f718b7)


## Creating a Role

To create a new role, you can use the ansible-galaxy command:

``` sh
ansible-galaxy init sonarQube-setup 
```

## Folder Structure

![image](https://github.com/user-attachments/assets/3424fd07-fd9b-464c-a0c3-c3dd526eb823)



## Variables


| **Variable**            | **Value**                | **Description**                                |  
|-------------------------|-------------------------|------------------------------------------------|  
| `postgresql_version`    | `"16"`                   | PostgreSQL version for SonarQube database.     |  
| `jdk_version`          | `"17"`                   | Java Development Kit version required.         |  
| `sonarqube_version`    | `"10.0.0.68432"`         | SonarQube version to be installed.             | 
| `sonarqube_web_port`   |  `"9000"`                | SonarQube webserver port |
| `sonarqube_user`       | `"ddsonar"`              | User for running SonarQube.                    |  
| `sonarqube_group`      | `"ddsonar"`              | Group for SonarQube user.                      |  
| `sonarqube_db`         | `"ddsonarqube"`          | Database name for SonarQube.                   |  
| `sonarqube_db_user`    | `"ddsonar"`              | Database user for SonarQube.                   |  
| `sonarqube_db_password` | `"mwd#2%#!!#%rgs"`     | Password for the SonarQube database user.      |  


## Subtasks Files
- This file is included in the `dependence.yml`, `dboperation.yml` & `sonarqube.yml` files.

    1. `dependence.yml`:- This Ansible playbook updates the package cache, installs required packages (using a variable required_packages), and ensures the PostgreSQL service is running and enabled at boot. 
  
    2. `dboperation.yml`:-This Ansible playbook ensures a PostgreSQL user and database for SonarQube exist. It creates the user if not present, updates the password, creates the database if missing, and grants full privileges to the user. 

    3. `sonarqube.yml`:-This Ansible playbook installs and configures SonarQube by downloading and extracting it, creating a dedicated user and group, setting permissions, configuring SonarQube using templates, setting up a systemd service, and updating sysctl settings.



## Templates for Configuration

We need to create two jinja2 templates :



- `sonarqube.conf.j2` teamplate includes parameteters to configure SonarQube database and webserver

- `sonarqube.service.j2` template creates a service file for setting up sonarqube.service


## Run playbook

``` sh
ansible-playbook -i <inventory-file> <playbook-name>
```

## Conclusion
Ansible Roles offers a modular approach to automate infrastructure, making deployments efficient, consistent, and less error-prone.
This guide illustrates the process of deploying SonarQube in a development environment through Ansible. By adhering to these instructions, you can effectively provision and set up SonarQube.



##  Contact Information


| **Name**       | **Email Address**        |
|----------------|--------------------------|
| Aayush Verma   | <aayush.verma@mygurukulam.co> |


## References

| **Link** | **Description** |
|------------------------------------------------------|------------------|
| [Documentaion link](https://medium.com/@sanyal.s271/introduction-to-sonarqube-elevate-your-code-quality-and-security-1c42fd092bdb) | SonarQube  |
| [SonarQube Installation link](https://medium.com/@deshdeepakdhobi/how-to-install-and-configure-sonarqube-on-aws-ec2-ubuntu-22-04-c89a3f1c2447)| SonarQube Installation |
