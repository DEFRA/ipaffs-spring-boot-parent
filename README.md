# Spring Boot Parent

## Introduction
This repository contains the parent POM for Java based IPAFFS repositories, it extends the POM provided by Spring Boot

## Secret scanning
Secret scanning is setup using [truffleHog](https://github.com/trufflesecurity/truffleHog).
It is used as a pre-push hook and will scan any local commits being pushed

### Pre-push hook setup
1. Install [truffleHog](https://github.com/trufflesecurity/truffleHog)
    - `brew install trufflesecurity/trufflehog/trufflehog`
2. Set DEFRA_WORKSPACE env var (`export DEFRA_WORKSPACE=/path/to/workspace`)
3. Potentially there's an older version of Trufflehog located at: `/usr/local/bin/trufflehog`. If so, remove this.
4. Run `mvn install` to configure hooks

### Steps to integrate
Include the following in the pom.xml of your spring boot service

```
<parent>
  <groupId>uk.gov.defra.tracesx</groupId>
  <artifactId>spring-boot-parent</artifactId>
  <version>desired version</version>
</parent>
```