# Driverless AI Deployment Templates

This repository contains different deployment templates for Driverless AI (DAI) scorers.

The structure is as follows:
* `common`: Code shared by multiple deployment templates
  * `swagger`: The shared REST API definitions. Please don't duplicate.
  * `rest-{framework}-model/api`: Modules with Java classes autogenerated from the API Swagger definition.
  * `transform`: Shared transformers between mojo and API classes.
* `{target}-scorer`: One module per deployment target.


## Build

This is a Gradle multi-project repository.
To get the resulting distribution archive, run:

```bash
$ ./gradlew distributionZip
```

The result of which is `./build/dai-deployment-templates-{CURRENT_VERSION}.zip`, which is in turn integrated in
the DAI build and deployment process.

Note that each of the templates is expected to inject its files in this archive in their respective gradle files.
Please see and follow examples in the existing deployment templates.


## Release

The built deployment templates zip archive is stored in S3 here: 
https://s3.console.aws.amazon.com/s3/buckets/artifacts.h2o.ai/releases/ai/h2o/dai-deployment-templates.

The push is handled by Jenkins from the `master` branch whenever the version in `gradle.properties`
does not contain the `-SNAPSHOT` suffix. Beware that Jenkins is happy to overwrite the artifact
if the version is not changed back to contain `-SNAPSHOT` again.

In addition to this, we maintain GitHub releases that mirror the released artifacts. For example
an artifact versioned `0.0.5` is tagged as `v0.0.5` with the actual release called
`Release v0.0.5`. This step is not automated.


### Upgrading Mojo2 Runtime

To upgrade the mojo2 runtime dependency version, just edit the corresponding line in the
`gradle.properties` file a push a new version of the deployment templates out as described
above.

Note that in order to be able to build against the new mojo2 runtime, the mojo2 runtime
implementation and the api jars have to be available in the H2O local Nexus:
http://172.17.0.53:8081/nexus. The mojo2 api also goes to the public
Maven repository: https://mvnrepository.com/artifact/ai.h2o/mojo2-runtime-api and it is
always a good idea to make sure it gets there for consistency. If this step fails,
it usually stops the whole release before fully pushing to the local Nexus.

Both the steps are handled by the `DAI Build: mojo2` Jenkins pipeline:
http://mr-0xc1:8080/view/H2OAI/job/mojo2/job/master/
if the `doRelease` parameter is checked when clicking on `Build with Parameters` on
the `master` branch (again, the master should currently contain a version without
the `-SNAPSHOT` suffix).

Note that there is an extra step to push the `mojo2-runtime-api` to the public Maven repo.
One has to login to Sonatype using H2o public nexus credentials, manually `close`
the appropriate staging repository with the new artifacts (usually the first one called
`aih2o.*`, but check the contents first!), and then `release` it.
The Maven UI takes time to display the new version, but the artifacts themselves usually
appear in the order of seconds/minutes after the release (i.e., you will not see the new
version on https://mvnrepository.com/artifact/ai.h2o/mojo2-runtime-api, but you will be
able to build against it).
