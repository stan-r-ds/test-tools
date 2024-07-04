## Dumping logs of test containers and making them available on github runner page:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Dumps logs of the given Docker containers to
 * {@link DumpTestContainersLogsExtension#LOGS_ROOT_DIR} after test annotated with this annotation
 * finishes. Sample usage:
 * <pre>
 *   {@literal}@Test
 *   {@literal}@DumpLogsOfContainers(containerNames = {
 *       "postgres",
 *       "mongo",
 *       "rabbit"
 *   })
 *   void shouldDoSomethingOnTestContainers() {
 *     // test code
 *   }
 * </pre>
 *
 * The logs can also be made accessible after a GitHub actions build finishes.
 * To achieve this, add the following step to the build's yaml file:
 * <pre>
 *      - name: Make module1's container test logs available
 *        uses: actions/upload-artifact@v4
 *        with:
 *          name: module1-container-test-logs
 *          path: module1/target/container-test-logs
 * </pre>
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface DumpLogsOfContainers {

  String[] containerNames();
}
```

```java
import java.io.IOException;
import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Collections;
import java.util.List;
import java.util.Optional;
import org.junit.jupiter.api.extension.AfterEachCallback;
import org.junit.jupiter.api.extension.ExtensionContext;

public class DumpTestContainersLogsExtension implements AfterEachCallback {

  public static final String LOGS_ROOT_DIR = "target/container-test-logs";

  @Override
  public void afterEach(ExtensionContext context) throws Exception {
    Method testMethod = context.getTestMethod().orElseThrow();
    List<String> containerNames = getContainerNames(testMethod);

    if (!containerNames.isEmpty()) {
      Path logFilesDir = prepareLogFilesDir(testMethod);
      for (String containerName : containerNames) {
        dumpContainerLogs(containerName, logFilesDir);
      }
    }
  }

  private static List<String> getContainerNames(Method testMethod) {
    return Optional
        .ofNullable(testMethod.getAnnotation(DumpLogsOfContainers.class))
        .map(DumpLogsOfContainers::containerNames)
        .map(List::of)
        .orElseGet(Collections::emptyList);
  }

  private static Path prepareLogFilesDir(Method testMethod) throws IOException {
    Path logFilesDir = Path.of(LOGS_ROOT_DIR, testMethod.getName());
    if (Files.exists(logFilesDir)) {
      Files.delete(logFilesDir);
    }
    Files.createDirectories(logFilesDir);

    return logFilesDir;
  }

  private static void dumpContainerLogs(String containerName, Path logFilesDir) throws Exception {
    Path logFile = logFilesDir.resolve(containerName + ".txt");
    Files.createFile(logFile);

    String[] command = {"docker", "logs", containerName};
    new ProcessBuilder(command)
        .redirectOutput(logFile.toFile())
        .start()
        .waitFor();
  }
}
```

