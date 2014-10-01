Apache TomEE + Shrinkwrap == JavaEE Boot
============

Based on this [article](http://java.dzone.com/articles/apache-tomee-shrinkwrap-javaee) published in DZone by @lordofthejars

```xml
<dependencies>
  <dependency>
    <groupId>org.apache.openejb</groupId>
    <artifactId>tomee-embedded</artifactId>
    <version>1.7.1</version>
  </dependency>

  <dependency>
    <groupId>org.apache.openejb</groupId>
    <artifactId>openejb-cxf-rs</artifactId>
    <version>4.7.1</version>
  </dependency>
  
  <dependency>
    <groupId>org.apache.openejb</groupId>
    <artifactId>tomee-jaxrs</artifactId>
    <version>1.7.1</version>
  </dependency>
  
  <dependency>
    <groupId>org.jboss.shrinkwrap</groupId>
    <artifactId>shrinkwrap-depchain</artifactId>
    <version>1.2.2</version>
    <type>pom</type>
  </dependency>
</dependencies>
```

```java
import javax.ejb.Stateless;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

@Stateless
@Path("/sample")
public class SampleController {

    @GET
    @Produces("text/plain")
    public String sample() {
        return "Hello World";
    }

    public static void main(String args[]) {
        TomEEApplication.run(HelloWorldServlet.class, SampleController.class);
    }
}
```

```java
public class TomEEApplication {

  private static void startAndDeploy(Archive<?> archive) {

    Container container;

      try {
        Configuration configuration = new Configuration();
        String tomeeDir = Files.createTempDirectory("apache-tomee").toFile().getAbsolutePath();
        configuration.setDir(tomeeDir);
        configuration.setHttpPort(8080);

        container = new Container();
        container.setup(configuration);

        final File app = new File(Files.createTempDirectory("app").toFile().getAbsolutePath());
        app.deleteOnExit();

        File target = new File(app, "app.war");
        archive.as(ZipExporter.class).exportTo(target, true);
        container.start();

        container.deploy("app", target);
        container.await();

      } catch (Exception e) {
          throw new IllegalArgumentException(e);
      }

      registerShutdownHook(container);

  }

  private static void registerShutdownHook(final Container container) {
    Runtime.getRuntime().addShutdownHook(new Thread() {
      @Override
      public void run() {
        try {
          if(container != null) {
            container.stop();
          }
        } catch (final Exception e) {
          throw new IllegalArgumentException(e);
        }
      }
    });
  }

  public static void run(Class<?> ... clazzes) {
    run(ShrinkWrap.create(WebArchive.class).addClasses(clazzes));
  }

  public static void run(WebArchive archive) {
    startAndDeploy(archive);
  }
}
```