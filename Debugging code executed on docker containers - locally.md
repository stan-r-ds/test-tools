In the Dockerfile, add code to enable debugging port:
```Dockerfile
ENV JAVA_TOOL_OPTIONS='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:5051'
EXPOSE 5051
```

In IntelliJ debug configuration of your test code, add VM option:
 - -Dmy-container-name-debug-port=5051

When creating a container in main code, add:
```java
  private void setupDebugPortIfPassedWithSystemProperty(String containerName) {
    String propertyName = containerName + "-debug-port";
    Integer debugPort = Integer.getInteger(propertyName);
    if (debugPort != null) {
      addExposedPort(debugPort);
      addFixedExposedPort(debugPort, debugPort);
    }
  }
```

In IntelliJ, configure a new debug configuration, to be able to attach to running container
 - create new Remote JVM Debug configuration
 - select "Attach to remote JVM"
 - host: localhost
 - port: the debug port from Dockerfile (for example 5051)

Now at any time when the container is running, you can launch the Remove JVM Debug and stop at breakpoints.
