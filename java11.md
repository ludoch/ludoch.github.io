# AppEngine Java 11 Runtime FAQ

The following questions apply to the AppEngine Java 11 standard runtime.

## What are the changes between the Java 8 and Java 11 standard runtimes?

Java11 is a [second gen](https://cloud.google.com/appengine/docs/standard/runtimes) App Engine runtime, and does not support the App Engine APIs (`com.google.appengine.api.*`).
 Instead, you need to use the [Cloud Java APIs](https://github.com/googleapis/google-cloud-java).

 Java11 must be configured
 with an app.yaml file instead of an appengine-web.xml file.
 Java11 can execute any Java framework, as long as you package a
 Web Server listening to port 8080 or $PORT env variable value.
 For example, Java11 can run a Spring Boot UberJAR as is.
 Java11 can support WAR format, but you need to package your
 Jetty/Tomcat or other Java web servers with your application. We have some samples on
 how this is done on our GitHub samples site.

## Running Kotlin apps with the AppEngine Java 11 runtime

You can run Kotlin apps on the AppEngine Java 11 runtime. You can find
Kotlin code samples in the
[GoogleCloudPlatform GitHub repository](https://github.com/GoogleCloudPlatform/java-docs-samples/tree/master/appengine-java11/kotlin-ktor).

## Using IntelliJ IDEA with AppEngine

You can IntelliJ IDEA Community and Ultimate Editions to develop
and deploy apps to AppEngine. You can use IntelliJ IDEA Community by
using the [Maven integration](/appengine/docs/standard/java/tools/using-maven).


## What is the minimal App Engine configuration file?

Along with your app, which can be a single JAR file, you can provide a 1 line
`app.yaml` configuration file:

<pre class="prettyprint lang-yaml">
runtime: java11
</pre>

You can use [Cloud SDK](/sdk) to generate that `app.yaml` file automatically by
using the following command:

<pre class="lang-sh prettyprint">
gcloud app deploy myjar.jar
</pre>

In this case, the `myjar.jar` must have a `Main-Class` entry in the manifest,
and all the classpath entries defined in the manifest `Class-Path:`
must be in the correct relative location from the main myjar.jar. These relative
dependencies will be deployed with the jar.

For a Maven project, the standard location for the app.yaml is under
`src/main/appengine` directory. The App Engine Maven plugin will
create a correct `target/appengine-staging` directory containing your jar
artifacts and this `app.yaml` file, ready for deployment.

## Can I use the Google Cloud Debugger?

Yes, when you do not define an entrypoint in `app.yaml`, this is the one that is generated automatically:

<pre class="prettyprint lang-yaml">
entrypoint: java -agentpath:/opt/cdbg/cdbg_java_agent.so=--log_dir=/var/log -jar yourjar.jar
</pre>

If you define your own Java entrypoint in app.yaml, you will have to add these flags:

<pre class="lang-sh prettyprint">
-agentpath:/opt/cdbg/cdbg_java_agent.so=--log_dir=/var/log
</pre>

 to your java command to use the Cloud Debugger.


## Using the Google Cloud Profiler

The Google Cloud Profiler is not automatically enabled, but the agent is provided in
the runtime environment. You can use the Google Cloud Profiler by adding these Java
flags:

<pre class="lang-sh prettyprint">
 -agentpath:/opt/cprof/profiler_java_agent.so=--logtostderr
</pre>

The profiler agent is located under `/opt/cprof/profiler_java_agent.so`.

## Which Open JDK environment is used?

AppEngine runs Java 11 apps run in a container secured by gVisor on an
up-to-date Ubuntu 18.04 Linux distribution and its supported openjdk-11-jdk
package.

AppEngine maintains the base image and updates the OpenJDK 11 package,
without a need to redeploy your application. The app is available in the /srv
directory.

## Serving static files

You can serve static files by using the `static_files` entry in your `app.yaml`
configration file. For example, to serve a Spring Boot application, you can do
the following steps before deployment:

1. Unpack the jar that contains all your app's classes (also known as a fat jar).

1. Add the following entrypoint in your `app.yaml` configuration file:

<pre class="prettyprint" suppresswarning="true">
entrypoint: java -noverify BOOT-INF/resources/:BOOT-INF/classes/:BOOT-INF/lib/* com.mycompany.myapp.YOUAPPCLASSNAME
</pre>

You need to add the following to your `app.yaml` file to
configure static resources when using an unpacked fat jar.

<pre class="prettyprint lang-yaml">
handlers:
- url: (/.*)
  static_files: BOOT-INF/classes/static\1
  upload: __NOT_USED__
  require_matching_file: True
  login: optional
  secure: optional
- url: /.*
  script: unsused
- url: /.*/
  script: unused
  login: optional
  secure: optional
- url: .*
  script: unused
  login: optional
  secure: optional
</pre>

This will serve all static files that are located in the
BOOT-INF/classes/static directory.

## Removing the port 8080 warning in app logs

If you see the following warnings in your app's log files:

<pre class="prettyprint" suppresswarning="true">
"App is listening on port 8080, it should instead listen on the port defined by the PORT environment variable. As a consequence, nginx cannot be started. Performance may be degraded. Please listen on the port defined by the PORT environment variable."
</pre>

Or:

<pre class="prettyprint" suppresswarning="true">
"App is listening on port 8080. We recommend your app listen on the port defined by the PORT environment variable to take advantage of an NGINX layer on port 8080."
</pre>

This log entry is created because your web application is listening to the hardcoded 8080 port
instead of a $PORT environment variable provided at execution. By using
the $PORT environment variable, which is 8081, AppEngine runs an NGINX
layer that can do HTTP response compression.

## Using GraalVM executables

The AppEngine standard Java 11 runtime supports GraalVM executables.
Once you have compiled your Java 11 app using GraalVM, you can use the
`entrypoint` setting in your `app.yaml` file to point to the executable.

For example, an executable with the filename `myexecutable` could have the
following`app.yaml` configuration file:

<pre class="prettyprint lang-yaml">
runtime: java11
entrypoint: ./myexecutable
</pre>

Note: some Google Cloud client libraries need extra GraalVM configuration in
order to work correctly. For more information, see
[GraalVM agent configuration](https://github.com/oracle/graal/blob/master/substratevm/CONFIGURE.md).

## Using Java frameworks with the Java 11 runtime

We have added [code samples](https://github.com/GoogleCloudPlatform/java-docs-samples/tree/master/appengine-java11)
for SpringBoot, Helidon, Quarkus, Micronaut, Jetty, SparkJava, Vert.x, and
Kotlin Ktor.

You are not limited to these frameworks and are encouraged to try your
preferred one, like Grails, Blade, Play!, Vaadin or
[jHipster](https://www.jhipster.tech/gcp/).

## Traffic splitting between Java 8 and Java 11 apps

AppEngine supports traffic splitting within each service version
between Java 8 and Java 11 runtimes. You can find out more about traffic
splitting by seeing [Splitting traffic](/appengine/docs/standard/java11/splitting-traffic).


## What Metadata server features are supported in the AppEngine Java 11 runtime?

The AppEngine Java 11 runtime supports the following
[Metadata server](/compute/docs/storing-retrieving-metadata) urls:

    /computeMetadata/v1/project/numeric-project-id
    /computeMetadata/v1/project/project-id
    /computeMetadata/v1/instance/zone
    /computeMetadata/v1/instance/service-accounts/default/aliases
    /computeMetadata/v1/instance/service-accounts/default/email
    /computeMetadata/v1/instance/service-accounts/default/scopes
    /computeMetadata/v1/instance/service-accounts/default/token
    /computeMetadata/v1/instance/service-accounts/{account}/aliases
    /computeMetadata/v1/instance/service-accounts/{account}/email
    /computeMetadata/v1/instance/service-accounts/{account}/scopes
    /computeMetadata/v1/instance/service-accounts/{account}/token
    /computeMetadata/v1beta1/project/numeric-project-id
    /computeMetadata/v1beta1/project/project-id
    /computeMetadata/v1beta1/instance/zone
    /computeMetadata/v1beta1/instance/service-accounts/default/aliases
    /computeMetadata/v1beta1/instance/service-accounts/default/email
    /computeMetadata/v1beta1/instance/service-accounts/default/scopes
    /computeMetadata/v1beta1/instance/service-accounts/default/token
    /computeMetadata/v1beta1/instance/service-accounts/{account}/aliases
    /computeMetadata/v1beta1/instance/service-accounts/{account}/email
    /computeMetadata/v1beta1/instance/service-accounts/{account}/scopes
    /computeMetadata/v1beta1/instance/service-accounts/{account}/token

