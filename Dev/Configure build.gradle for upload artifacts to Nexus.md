# Configure build.gradle for upload artifacts to Nexus


build.gradle
```groovy
publishing {
    publications {
        nexus(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            url "${repositoryReleaseUrl}"
            credentials {
                username "${repositoryUser}"
                password "${repositoryPassword}"
            }
        }
    }
}

```

```properties
group=com.github
version=0.1.0

repositoryUrl=${repositoryUrl}

repositoryUser=${repositoryUser}
repositoryPassword=${repositoryPassword}
repositoryReleaseUrl=http://nexus.ru/repository/maven-releases/

```

