# Blue Falcon - Fork Syslor

Fork de [Reedyuk/blue-falcon](https://github.com/Reedyuk/blue-falcon) publié sur GitLab Packages pour usage interne.

## Coordonnées Maven

```
com.syslor.bluefalcon:blue-falcon:2.5.0
```

## Pré-requis

### 1. Créer un Deploy Token GitLab

Settings > Repository > Deploy Tokens :
- Scope : `write_package_registry` (pour publier)
- Créer un second token avec `read_package_registry` (pour consommer)

### 2. Configurer `local.properties`

```properties
gitlabProjectId=<PROJECT_ID>
gitlabDeployUsername=<deploy-token-username>
gitlabDeployPassword=<deploy-token-password>
```

Le Project ID se trouve dans Settings > General sur GitLab.

En CI/CD, ces valeurs sont remplacées automatiquement par `CI_PROJECT_ID` et `CI_JOB_TOKEN`.

## Publier

```bash
# Toutes les plateformes (Android, iOS, macOS, JS, KMP metadata)
./gradlew publishAllPublicationsToGitLabRepository

# Une plateforme spécifique
./gradlew publishAndroidReleasePublicationToGitLabRepository
./gradlew publishIosArm64PublicationToGitLabRepository

# En local (sans credentials)
./gradlew publishToMavenLocal
```

## Consommer dans un projet

### build.gradle.kts (repositories)

```kotlin
repositories {
    maven {
        name = "GitLab"
        url = uri("https://gitlab.com/api/v4/projects/<PROJECT_ID>/packages/maven")
        credentials {
            username = gitlabDeployUser  // deploy token avec read_package_registry
            password = gitlabDeployPass
        }
    }
}
```

### build.gradle.kts (dependencies)

```kotlin
commonMain.dependencies {
    implementation("com.syslor.bluefalcon:blue-falcon:2.5.0")
}
```

## Différences avec upstream

| | Upstream | Fork |
|---|---|---|
| GroupId | `dev.bluefalcon` | `com.syslor.bluefalcon` |
| Registry | Maven Central | GitLab Packages (privé) |
| PRs mergées | #181, #182, #183 | incluses |
