# Guides
There are many ways you can contribute to the SkyWalking community.

- Go through our documents, and point out or fix a problem. Translate the documents into other languages.
- Download our [releases](http://skywalking.apache.org/downloads/), try to monitor your applications, and provide feedback to us.
- Read our source codes. For details, reach out to us.
- If you find any bugs, [submit an issue](https://github.com/apache/skywalking/issues). You can also try to fix it.
- Find [help wanted issues](https://github.com/apache/skywalking/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22). This is a good place for you to start.
- Submit an issue or start a discussion at [GitHub issue](https://github.com/apache/skywalking/issues/new).
- See all mail list discussions at [website list review](https://lists.apache.org/list.html?dev@skywalking.apache.org).
If you are already a SkyWalking committer, you can log in and use the mail list in the browser mode. Otherwise, subscribe following the step below.
- Issue reports and discussions may also take place via `dev@skywalking.apache.org`. 
Mail to `dev-subscribe@skywalking.apache.org`, and follow the instructions in the reply to subscribe to the mail list. 

## Contact Us
All of the following channels are open to the community.
* Submit an [issue](https://github.com/apache/skywalking/issues)
* Mail list: **dev@skywalking.apache.org**. Mail to `dev-subscribe@skywalking.apache.org`. Follow the instructions in the reply to subscribe to the mail list.
* [Gitter](https://gitter.im/openskywalking/Lobby)
* QQ Group: 392443393

## Become an official Apache SkyWalking Committer
The PMC assesses the contributions of every contributor, including their code contributions. It also  promotes, votes on, and invites new committers and PMC members according to the Apache guides.
See [Become official Apache SkyWalking Committer](asf/committer.md) for more details.

## For code developer
For developers, the starting point is the [Compiling Guide](How-to-build.md). It guides developers on how to build the project in local and set up the environment.

### Integration Tests
After setting up the environment and writing your codes, to facilitate integration with the SkyWalking project, you'll
need to run tests locally to verify that your codes would not break any existing features,
as well as write some unit test (UT) codes to verify that the new codes would work well. This will prevent them from being broken by future contributors.
If the new codes involve other components or libraries, you should also write integration tests (IT).

SkyWalking leverages the plugin `maven-surefire-plugin` to run the UTs and uses `maven-failsafe-plugin`
to run the ITs. `maven-surefire-plugin` excludes ITs (whose class name starts with `IT`)
and leaves them for `maven-failsafe-plugin` to run, which is bound to the `verify` goal and `CI-with-IT` profile.
Therefore, to run the UTs, try `./mvnw clean test`, which only runs the UTs but not the ITs.

If you would like to run the ITs, please activate the `CI-with-IT` profile
as well as the the profiles of the modules whose ITs you want to run.
E.g. if you would like to run the ITs in `oap-server`, try `./mvnw -Pbackend,CI-with-IT clean verify`,
and if you would like to run all the ITs, simply run `./mvnw -Pall,CI-with-IT clean verify`.

Please be advised that if you're writing integration tests, name it with the pattern `IT*` so they would only run with the `CI-with-IT` profile.

### End to End Tests (E2E)
Since version 6.3.0, we have introduced more automatic tests to perform software quality assurance. E2E is an integral part of it.

> End-to-end testing is a methodology used to test whether the flow of an application is performing as designed from start to finish.
 The purpose of carrying out end-to-end tests is to identify system dependencies and to ensure that the right information is passed between various system components and systems.

The E2E test involves some/all of the OAP server, storage, coordinator, webapp, and the instrumented services, all of which are orchestrated by `docker-compose` or `KinD`. 
Since version 8.9.0, we immigrate to e2e-v2 which leverage [skywalking-infra-e2e](https://github.com/apache/skywalking-infra-e2e) and [skywalking-cli](https://github.com/apache/skywalking-cli) to do the whole e2e process.
`skywalking-infra-e2e` is used to control the e2e process and `skywalking-cli` is used to interact with the OAP such as request and get response metris from OAP.

#### Writing E2E Cases

- Set up the environment
1. Set up `skywalking-infra-e2e`
1. Set up `skywalking-cli`, `yq` (generally these 2 are enough) and others tools if your cases need. Can reference the script under `skywalking/test/e2e-v2/script/prepare/setup-e2e-shell`.

- Orchestrate the components

The goal of the E2E tests is to test the SkyWalking project as a whole, including the OAP server, storage, coordinator, webapp, and even the frontend UI (not for now), on the single node mode as well as the cluster mode. Therefore, the first step is to determine what case we are going to verify, and orchestrate the 
components.
 
To make the orchestration process easier, we're using a [docker-compose](https://docs.docker.com/compose/) that provides a simple file format (`docker-compose.yml`) for orchestrating the required containers, and offers an opportunity to define the dependencies of the components.

Follow these steps:
1. Decide what (and how many) containers will be needed. For example, for cluster testing, you'll need > 2 OAP nodes, coordinators (e.g. zookeeper), storage (e.g. ElasticSearch), and instrumented services;
1. Define the containers in `docker-compose.yml`, and carefully specify the dependencies, starting orders, and most importantly, link them together, e.g. set the correct OAP address on the agent end, and set the correct coordinator address in OAP, etc.
1. Define the e2e case [config](https://skywalking.apache.org/docs/skywalking-infra-e2e/latest/en/setup/configuration-file/) in `e2e.yaml`.
1. Write the expected data(yml) for verify.

- [Run e2e test](https://skywalking.apache.org/docs/skywalking-infra-e2e/latest/en/setup/run-e2e-tests/)

All e2e cases should under `skywalking/test/e2e-v2/cases`. You could execute e2e run command in `skywalking/` e.g.
```
e2e run -c test/e2e-v2/cases/alarm/h2/e2e.yaml
```

- Troubleshooting

We expose all logs from all containers to the stdout in the non-CI (local) mode, but save and upload them to the GitHub server. You can download them (only when the tests have failed) at "Artifacts/Download artifacts/logs" (see top right) for debugging.

**NOTE:** Please verify the newly-added E2E test case locally first. However, if you find that it has passed locally but failed in the PR check status, make sure that all the updated/newly-added files (especially those in the submodules)
are committed and included in the PR, or reset the git HEAD to the remote and verify locally again.

### Project Extensions
The SkyWalking project supports various extensions of existing features. If you are interesting in writing extensions,
read the following guides.

This guides you in developing SkyWalking agent plugins to support more frameworks. Developers for both open source and private plugins should read this. 
- If you would like to build a new probe or plugin in any language, please read the [Component library definition and extension](Component-library-settings.md) document.
- [Storage extension development guide](storage-extention.md). Potential contributors can learn how to build a new 
storage implementor in addition to the official one.
- Customize analysis using OAL scripts. OAL scripts are located in `config/oal/*.oal`. You could modify them and reboot the OAP server. Read 
[Observability Analysis Language Introduction](../concepts-and-designs/oal.md) to learn more about OAL scripts.
- [Source and scope extension for new metrics](source-extension.md). For analysis of a new metric which SkyWalking
hasn't yet provided. Add a new receiver, rather than choosing an [existing receiver](../setup/backend/backend-receivers.md).
You would most likely have to add a new source and scope. To learn how to do this, read the document.

### UI developer
Our UI consists of static pages and the web container.

- [RocketBot UI](https://github.com/apache/skywalking-rocketbot-ui) is SkyWalking's primary UI since the 6.1 release.
It is built with vue + typescript. Learn more at the rocketbot repository.
- **Web container** source codes are in the `apm-webapp` module. This is a simple zuul proxy which hosts
static resources and sends GraphQL query requests to the backend.
- [Legacy UI repository](https://github.com/apache/skywalking-ui) is retained, but not included
in SkyWalking releases since 6.0.0-GA.

### OAP backend dependency management
> This section is only applicable to dependencies of the backend module.

As one of the Top Level Projects of The Apache Software Foundation (ASF), SkyWalking must follow the [ASF 3RD PARTY LICENSE POLICY](https://apache.org/legal/resolved.html). So if you're adding new dependencies to the project, you should make sure that the new dependencies would not break the policy, and add their LICENSE and NOTICE to the project.

We have a [simple script](../../../tools/dependencies/check-LICENSE.sh) to help you make sure that you haven't missed out any new dependencies:
- Build a distribution package and unzip/untar it to folder `dist`.
- Run the script in the root directory. It will print out all new dependencies.
- Check the LICENSE and NOTICE of those dependencies to make sure that they can be included in an ASF project. Add them to the `apm-dist/release-docs/{LICENSE,NOTICE}` file.
- Add the names of these dependencies to the `tools/dependencies/known-oap-backend-dependencies.txt` file (**in alphabetical order**). `check-LICENSE.sh` should pass in the next run.

## Profile
The performance profile is an enhancement feature in the APM system. We use thread dump to estimate the method execution time, rather than adding multiple local spans. In this way, the cost would be significantly reduced compared to using distributed tracing to locate the slow method. This feature is suitable in the production environment. The following documents are key to understanding the essential parts of this feature.
- [Profile data report protocol](https://github.com/apache/skywalking-data-collect-protocol/tree/master/profile) is provided through gRPC, just like other traces and JVM data.
- [Thread dump merging mechanism](backend-profile.md) introduces the merging mechanism. This mechanism helps end users understand profile reports.
- [Exporter tool of profile raw data](backend-profile-export.md) guides you on how to package the original profile data for issue reports when the visualization doesn't work well on the official UI.

## Release
If you're a committer, read the [Apache Release Guide](How-to-release.md) to learn about how to create an official Apache version release in accordance with avoid Apache's rules. As long as you keep our LICENSE and NOTICE, the Apache license allows everyone to redistribute.
