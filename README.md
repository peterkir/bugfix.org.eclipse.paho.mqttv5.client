# bugfix.org.eclipse.paho.mqttv5.client

A minimal bnd workspace that produces an OSGi wrapper bundle for the Maven artifact
`org.eclipse.paho:org.eclipse.paho.mqttv5.client:1.2.5`.

## Published coordinates

| Property     | Value                              |
|--------------|------------------------------------|
| `groupId`    | `io.klib.wrap`                     |
| `artifactId` | `org.eclipse.paho.mqttv5.client`   |
| `version`    | `1.2.5.99`                         |

## Structure

```
.
├── cnf/                          # bnd workspace configuration
│   ├── build.bnd                 # workspace-level build settings
│   ├── ext/
│   │   └── repositories.bnd     # Maven repository plugin (resolves paho from Central)
│   └── paho.maven               # MavenBndRepository index
├── org.eclipse.paho.mqttv5.client/
│   ├── bnd.bnd                  # OSGi wrapper bundle instructions
│   └── build.gradle             # Gradle project (maven-publish + signing)
├── settings.gradle              # bnd workspace plugin
├── gradlew / gradlew.bat        # Gradle wrapper (targets Gradle 8.12)
└── .github/
    ├── scripts/
    │   ├── ci-build.sh          # build + publish to dist/m2
    │   └── sonatype-upload.sh   # upload dist/m2 to Sonatype Central Portal
    └── workflows/
        └── build.yml            # GitHub Actions CI workflow
```

## Local build

```bash
# Build the wrapper bundle
./gradlew build

# Build and publish to dist/m2 (no signing)
./gradlew build publishMavenOsgiPublicationToDistRepository
```

The built JAR is at `org.eclipse.paho.mqttv5.client/generated/org.eclipse.paho.mqttv5.client.jar`.

The published Maven artifacts are placed under `dist/m2/`.

## Release / upload to Sonatype Central

### Prerequisites

Set the following GitHub repository secrets:

| Secret            | Description                                |
|-------------------|--------------------------------------------|
| `GPG_PRIVATE_KEY` | Armored GPG private key (base64 or plain)  |
| `GPG_PASSPHRASE`  | Passphrase for the GPG key                 |
| `GPG_KEY_ID`      | Key ID (long format, e.g. `ABCDEF01234567`) |
| `SONATYPE_BEARER` | Bearer token from Sonatype Central Portal  |

### CI workflow

The `CI` workflow (`.github/workflows/build.yml`) runs on every push and
`workflow_dispatch`. It:

1. Checks out the repository.
2. Runs `.github/scripts/ci-build.sh` which:
   - Imports and activates the GPG key if secrets are set.
   - Runs `./gradlew build publishMavenOsgiPublicationToDistRepository` (with
     signing when GPG secrets are present).
3. Runs `.github/scripts/sonatype-upload.sh dist/m2` when `SONATYPE_BEARER`
   is available, uploading the contents of `dist/m2` to Sonatype Central
   Portal as a `USER_MANAGED` deployment.

### Manual upload

```bash
export SONATYPE_BEARER=<your-token>
./.github/scripts/sonatype-upload.sh dist/m2
```

After upload the script prints the Deployment ID and stores it in
`dist/m2_DEPLOYMENTID.txt`. Log in to <https://central.sonatype.com> to
review and release the deployment manually.
