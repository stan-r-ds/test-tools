Option 1.
src/main/resources/application.properties:

quarkus.log.min-level=TRACE
quarkus.log.level=TRACE

---

Option 2 (deprecated)
```Dockerfile
ENV JAVA_OPTS_APPEND="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -Dquarkus.log.level=TRACE"
```

The last property sets the log level. The entries on the left are usual defaults.

Warning:
 - you may get in the service container logs: "Root log level TRACE set below minimum logging level DEBUG, promoting it to DEBUG. Set the build time configuration property 'quarkus.log.min-level' to 'TRACE' to avoid this warning"
 - fix: add "quarkus.log.min-level=TRACE" to src/main/application.properties
