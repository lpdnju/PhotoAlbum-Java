[← Back to aggregate report](../../index.md)

# photo-album

## Summary

| Metric | Value |
|--------|-------|
| Total Issues | 7 |
| Mandatory Blockers | 3 |
| Potential Issues | 4 |

## Application Information

| Property | Value |
|----------|-------|
| Language | Java |
| Frameworks | Spring Boot, Spring |
| Build tools | Maven |
| JDK version | 1.8 |

## Cloud Readiness Issues

| Issue Name | Criticality | Story Points | Occurrences |
|------------|-------------|--------------|-------------|
| Oracle database found | Potential | 8 | [6](#Oracle_database_found) |
| Password found in configuration file | Potential | 3 | [3](#Password_found_in_configuration_file) |
| Server port configuration found | Potential | 1 | [2](#Server_port_configuration_found) |
| Restricted configurations found | Potential | 2 | [2](#Restricted_configurations_found) |

### Issue Details

<details id="Oracle_database_found">
<summary><b>Oracle database found</b> — affected files</summary>

- `pom.xml (line 52)`
- `docker-compose.yml (line 32)`
- `src/main/resources/application-docker.properties (line 2)`
- `src/main/resources/application.properties (line 10)`
- `src/main/resources/application-docker.properties (line 5)`
- `src/main/resources/application.properties (line 13)`

</details>

<details id="Password_found_in_configuration_file">
<summary><b>Password found in configuration file</b> — affected files</summary>

- `src/test/resources/application-test.properties (line 5)`
- `src/main/resources/application-docker.properties (line 4)`
- `src/main/resources/application.properties (line 12)`

</details>

<details id="Server_port_configuration_found">
<summary><b>Server port configuration found</b> — affected files</summary>

- `src/main/resources/application-docker.properties (line 24)`
- `src/main/resources/application.properties (line 2)`

</details>

<details id="Restricted_configurations_found">
<summary><b>Restricted configurations found</b> — affected files</summary>

- `src/main/resources/application.properties (line 2)`
- `src/main/resources/application-docker.properties (line 24)`

</details>

## Upgrade Issues

| Issue Name | Criticality | Story Points | Occurrences |
|------------|-------------|--------------|-------------|
| Spring Boot Version is End of OSS Support | Mandatory | 8 | [7](#Spring_Boot_Version_is_End_of_OSS_Support) |
| Spring Framework Version End of OSS Support | Mandatory | 8 | [3](#Spring_Framework_Version_End_of_OSS_Support) |
| Legacy Java version | Mandatory | 5 | [3](#Legacy_Java_version) |

### Issue Details

<details id="Spring_Boot_Version_is_End_of_OSS_Support">
<summary><b>Spring Boot Version is End of OSS Support</b> — affected files</summary>

- `pom.xml (line 46)`
- `pom.xml (line 78)`
- `pom.xml (line 34)`
- `pom.xml (line 59)`
- `pom.xml (line 40)`
- `pom.xml (line 92)`
- `pom.xml (line 72)`

</details>

<details id="Spring_Framework_Version_End_of_OSS_Support">
<summary><b>Spring Framework Version End of OSS Support</b> — affected files</summary>

- `pom.xml (line 34)`
- `pom.xml (line 46)`
- `pom.xml (line 78)`

</details>

<details id="Legacy_Java_version">
<summary><b>Legacy Java version</b> — affected files</summary>

- `pom.xml (line 24)`
- `pom.xml (line 25)`
- `pom.xml (line 26)`

</details>

---

[Share feedback](https://aka.ms/ghcp-appmod/feedback)
