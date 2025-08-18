In the Dockerfile, add code to enable debugging port:
```Dockerfile
ENV JAVA_TOOL_OPTIONS='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:5051'
EXPOSE 5051
```

In `dev.streamx.runner.container.StreamxServiceContainer`, add:
```java
  private void initialize(Integer exposedHttpPort) {
    ...
    setupDebugPortIfPassedWithSystemProperty();
    ...
  }

  private void setupDebugPortIfPassedWithSystemProperty(String containerName) {
    String propertyName = containerName + "-debug-port";
    Integer debugPort = Integer.getInteger(propertyName);
    if (debugPort != null) {
      addExposedPort(debugPort);
      addFixedExposedPort(debugPort, debugPort);
    }
  }
```

Run streamx with:
    java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=\*:5051 -Dblueprint-web-debug-port=5051 -jar ../streamx/runner/target/quarkus-app/quarkus-run.jar mesh.yaml

In IntelliJ, configure a new debug configuration, to be able to attach to running container
 - create new Remote JVM Debug configuration
 - select "Attach to remote JVM"
 - host: localhost
 - port: the debug port from Dockerfile (for example 5051)

Now at any time when the container is running, you can launch the Remove JVM Debug and stop at breakpoints.
