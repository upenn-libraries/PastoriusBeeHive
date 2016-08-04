# Pastorius Bee Hive

This is adapted from Glenn Robson's
[SimpleAnnotationServer](https://github.com/glenrobson/SimpleAnnotationServer).
Thanks to [blabrit](https://github.com/blalbrit) for directing me to it.  It is
customized to serve images from the Penn Pastorius Beehive hosted via IIIF at
Stanford (again, thanks, [blabrit](https://github.com/blalbrit)).  It also adds FORM
authentication and changes `pom.xml` references to SimpleAnnotationServer to
PastoriusBeeHive.

Still it's mostly SimpleAnnotationServer with a changed list of manifests and
authentication configured. What follows are instructions for building and
deploying the Bee Hive app followed by the unedited SimpleAnnotationServer
Read Me text.

## Deploy

Clone the repo

```
$ git clone https://github.com/upenn-libraries/PastoriusBeeHive.git
```

Change the Jena `data_dir` location in the `pom.xml` to match your system:

```xml
        <init-param>
            <param-name>data_dir</param-name>
            <param-value>/path/to/beehive/data</param-value>
            <description>Sets the directory containing TDB RDF database</description>
        </init-param>
```


PastoriusBeeHive uses form authentication to restrict access to the site. It's set up
this way in `src/main/webapp/WEB-INF/web.xml`:

```xml
    <security-constraint>
        <web-resource-collection>
            <url-pattern>/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>admin</role-name>
        </auth-constraint>
    </security-constraint>
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Public</web-resource-name>
            <description>Matches a few special pages.</description>
            <url-pattern>/public/*</url-pattern>
        </web-resource-collection>
        <!-- No auth-constraint means everybody has access! -->
    </security-constraint>
    <login-config>
        <auth-method>FORM</auth-method>
        <realm-name>Test Realm</realm-name>
        <form-login-config>
            <form-login-page>/logon.html</form-login-page>
            <form-error-page>/logonError.html</form-error-page>
        </form-login-config>
    </login-config>
```

There must be a user defined in the Jetty base realm coniguration with the
role `admin`.

No `security-constraint` is placed on file in `public` directory.

There are two files added to the `webapp` directory for login and login
failure.

## Jetty configuration

The Jetty based directory should have a realm XML file in the `/etc`
directory. Here's a sample:

```xml
<!-- sample-realm.xml -->
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-" "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure id="Server" class="org.eclipse.jetty.server.Server">
    <!-- =========================================================== -->
    <!-- Configure Authentication Login Service                      -->
    <!-- Realms may be configured for the entire server here, or     -->
    <!-- they can be configured for a specific web app in a context  -->
    <!-- configuration (see $(jetty.home)/webapps/test.xml for an    -->
    <!-- example).                                                   -->
    <!-- =========================================================== -->
    <Call name="addBean">
      <Arg>
        <New class="org.eclipse.jetty.security.HashLoginService">
          <Set name="name">Beehive Realm</Set>
          <Set name="config"><Property name="beehive.realm" default="etc/realm.properties"/></Set>
          <Set name="refreshInterval">0</Set>
        </New>
      </Arg>
    </Call>
</Configure>
```

The realm should be loaded in the `/start.ini` file:

```ini
# /start.ini
# Create and configure the test realm
etc/sample-realm.xml
sample.realm=etc/realm.properties
```

Passwords for at least one user with the admin role should be defined in
`/etc/realm.properties`:

```properties
# /etc/realm.properties
#
# This file defines users passwords and roles for a HashUserRealm
#
# The format is
#  <username>: <password>[,<rolename> ...]
#
admin: CRYPT:adpexzg3FUZAk,admin
```

See default Jetty `demo-base/etc/realm.properties` for more information.

## Dumping the data annotation data

PastoriusBeeHive is set to use the default SimpleAnnotationServer RDF store
Jena (see below).  As pointed out in the SimpleAnnotationServer wiki page on
[using Sesame instead of Jena as annotation store] [Sesame], Jena does not
allow seprate processes to access it at the same time.  The webapp will need
to be taken down to dump the data.

[Sesame]: https://github.com/glenrobson/SimpleAnnotationServer/blob/master/doc/Sesame.md

To dump the data, use `tdbdump` from the [Apache Jena] [Jena] download:

[Jena]: https://jena.apache.org/download/index.cgi "Apache Jena downloads"

```
$ bin/tdbdump --loc=/path/to/beehive/data_dir

<http://dev.llgc.org.uk/annotation/1468002091431> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/oa#Annotation> <http://dev.llgc.org.uk/annotation/1468002091431> .
<http://dev.llgc.org.uk/annotation/1468002091431> <http://www.w3.org/ns/oa#motivatedBy> <http://www.w3.org/ns/oa#commenting> <http://dev.llgc.org.uk/annotation/1468002091431> .
<http://dev.llgc.org.uk/annotation/1468002091431> <http://www.w3.org/ns/oa#hasTarget> _:B6d48786bee1db820611f2d00c69669c5 <http://dev.llgc.org.uk/annotation/1468002091431> .
<http://dev.llgc.org.uk/annotation/1468002091431> <http://www.w3.org/ns/oa#hasBody> _:B8a9c4c7d589353334b32e432cfaadb62 <http://dev.llgc.org.uk/annotation/1468002091431> .
# ... etc. ...
```

# SimpleAnnotationServer

> NOTE: This fork of SimpleAnnotationServer has been altered to be built as a war
> file and dropped into a specific Jetty configuration.  If you want to run
> SimpleAnnotationServer as described below, you should probably goto its [Github
> page](https://github.com/glenrobson/SimpleAnnotationServer).

This is an Annotation Server which is compatible with [IIIF](http://iiif.io) and [Mirador](https://github.com/IIIF/mirador). This Annotation Server includes
a copy of Mirador so you can get started creating annotations straight away. The annotations are stored as linked data in an [Apache Jena](https://jena.apache.org/) triple store.

## Getting Started
**Requires:**
 * Java 1.7
 * [maven](https://maven.apache.org/)

To begin working with Mirador and the Simple Annotation Server do the following:

 * Download code

```git clone https://github.com/glenrobson/SimpleAnnotationServer.git```

 * Move into the SimpleAnnotationServer directory.

```cd SimpleAnnotationServer```

 * Start the jetty http server

```mvn jetty:run```

 * Start Annotating

Navigate to [http://localhost:8888/index.html](http://localhost:8888/index.html)

You should now see Mirador with the default example objects. You can choose any manifest to start annotating.

## Further guides

For further details on the SimpleAnnotationServer see:

 * [Adding your own Manifests](doc/NewManifests.md)
 * [Populating the Annotation Store with IIIF Annotation List](doc/PopulatingAnnotations.md)
 * [Developing Mirador with SimpleAnnotationServer](doc/DevGuide.md)
 * [Deploying to tomcat](doc/tomcat.md)
 * [Using the Sesame RDF store](doc/Sesame.md)
 * [Remote Annotation Store](doc/RemoteStore.md)

## Missing Functions

Note this project doesn't currently contain Authentication or Versioning of Annotations. If your looking for these functions have a look at [Triannon from Stanford](https://github.com/sul-dlss/triannon) or [Catch from Harvard](https://github.com/annotationsatharvard/catcha).

## Thanks

Thanks to:

 * [azaroth42](https://github.com/azaroth42) for help with JsonLd framing and other useful tips.
 * [Illtud](https://github.com/illtud) and [Paul](https://twitter.com/sankesolutions) for help with testing and fixing build problems.
 * [Dan](https://twitter.com/Surfrdan) for introducing me to Apache Jena

and finally thanks to the IIIF and Mirador communities which make all this cool stuff possible.
