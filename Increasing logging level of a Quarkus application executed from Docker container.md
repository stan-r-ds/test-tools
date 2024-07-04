```Dockerfile
ENV JAVA_OPTS_APPEND="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -Dquarkus.log.level=DEBUG"
```

The last property sets the log level. The entries on the left are usual defaults.
