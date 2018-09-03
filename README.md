# Example resolving dynamic versions with ivy on Sonatype Nexus 3

## introduction

Nexus 3 is missing a feature, to list artifacts or folders by just using suburls of the final artifact.
Nexus 2 had this feature and the project here is a minimal apache ant/ivy example to demonstrate effects of the changed behavior.

## problem

If ivy is resolving [dynamic versions](https://ant.apache.org/ivy/history/2.4.0/ivyfile/dependency.html) from a maven repository it usually reads a `maven-metadata.xml` file (which should 
contain all available versions).

The problem with ivy is, that itself does not create or update the `maven-metadata.xml`
([see this posting](https://stackoverflow.com/a/15797166/1756183)). This way one either have to rely on
[scheduled tasks](https://blog.sonatype.com/2009/09/nexus-scheduled-tasks/) of the repository manager, on the price of 
timing issues. New artifact versions may not be found immediately after being uploaded or `SNAPSHOT` versions are not 
correctly identified as changed and become obsolete in local caches.

The solution to this problem is to tell ivys ibiblio resolver to `useMavenMetadata="false"`
([see this blog](https://ssinghvi.wordpress.com/2012/01/26/unable-to-retrieve-latest-artifact-from-sonatype-nexus-using-apache-ivy/)).

## effect

Resolving dynamic versions - such as `commons-logging:commons-logging:1.+` - from Nexus 2 works just fine:
```
...
[ivy:resolve] == resolving dependencies #;working@NBDR246->commons-logging#commons-logging;1.+ [default->compile]
[ivy:resolve] default-cache: no cached resolved revision for commons-logging#commons-logging;1.+
[ivy:resolve]           tried https://repo1.maven.org/maven2/commons-logging/commons-logging/[revision]/commons-logging-[revision].pom
[ivy:resolve] Execution environment profile JavaSE-1.2 loaded
...
```

while it does not with a Nexus 3 repository manager:
```
...
[ivy:resolve] default-cache: no cached resolved revision for commons-logging#commons-logging;1.+
[ivy:resolve]           tried http://localhost:8081/repository/maven-public/commons-logging/commons-logging/[revision]/commons-logging-[revision].pom

[ivy:resolve]           tried http://localhost:8081/repository/maven-public/commons-logging/commons-logging/[revision]/commons-logging-[revision].jar

[ivy:resolve]   ibiblio: no ivy file nor artifact found for commons-logging#commons-logging;1.+
[ivy:resolve] WARN:     module not found: commons-logging#commons-logging;1.+
[ivy:resolve] WARN: ==== ibiblio: tried
[ivy:resolve] WARN:   http://localhost:8081/repository/maven-public/commons-logging/commons-logging/[revision]/commons-logging-[revision].pom
[ivy:resolve] WARN:   -- artifact commons-logging#commons-logging;1.+!commons-logging.jar:
[ivy:resolve] WARN:   http://localhost:8081/repository/maven-public/commons-logging/commons-logging/[revision]/commons-logging-[revision].jar
[ivy:resolve]   resolved ivy file produced in cache
[ivy:resolve] :: downloading artifacts ::
[ivy:resolve] :: resolution report :: resolve 85ms :: artifacts dl 0ms
        ---------------------------------------------------------------------
        |                  |            modules            ||   artifacts   |
        |       conf       | number| search|dwnlded|evicted|| number|dwnlded|
        ---------------------------------------------------------------------
        |      default     |   1   |   0   |   0   |   0   ||   0   |   0   |
        ---------------------------------------------------------------------
[ivy:resolve] WARN:     ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve] WARN:     ::          UNRESOLVED DEPENDENCIES         ::
[ivy:resolve] WARN:     ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve] WARN:     :: commons-logging#commons-logging;1.+: not found
[ivy:resolve] WARN:     ::::::::::::::::::::::::::::::::::::::::::::::
...
```

## running the example

The following command can be used to reproduce the described effects (a running ant installation is required, ivy will be downloaded):
```
# maven central url
ant resolve-dynamic-central
# local nexus 3 url (default installation)
ant resolve-dynamic-local
# individual repository base-url
ant resolve-dynamic -Divy.repository.url=<url>
```