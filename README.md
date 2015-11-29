# JavaPar
Parrallel



## Write the automated test 
```Java
import org.junit.Test;
import play.mvc.Http;
import play.mvc.Result;
import play.test.WithApplication;

import static org.junit.Assert.*;
import static play.test.Helpers.*;

public class HomeFeed2ControllerTest extends WithApplication {

    @Test
    public void testHomefeed() throws Exception {
        Http.RequestBuilder request = new Http.RequestBuilder()
                .method(GET)
                .uri("/myPath")
                .header("X-Customer-Id","4f11e0df-f127-4812-abc6-356b139693f4");

        Result result = route(request);
        assertEquals(OK, result.status());
        assertTrue(contentAsString(result).contains("likedBy"));
        assertTrue(contentAsString(result).contains("likedAt"));
  
        }
    }

``` 

#### Set configuration in InteliJ and set the VM options to e.g.:
``` 
-Dconfig.resource=local.conf
``` 
#### Otherwise, add these options to the build.sbt and  specify your mylocal.conf file

``` 
javaOptions in Test += "-Dconfig.file=conf/mylocal.conf"

javaOptions in Test ++= Seq(
  "-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9998",
  "-Xms512M",
  "-Xmx1536M",
  "-Xss1M",
  "-XX:MaxPermSize=384M"
)
```

## Search service optimization
There are many approaches to optimize  Cassandra queries and to run tasks in parallel.
### Example 1: Using Java futures

```java
....
private int threadNum = 2;
private ExecutorService executorService1 = Executors.newFixedThreadPool(threadNum);
Future<JsonArray> f1 = null;
Future<JsonArray> f1 = null;

f1 = executorService1.submit(new Callable<JsonArray>() {
    @Override
    public JsonArray call() throws Exception {
        return method1(a,b,c);
    }
});


f2 = executorService1.submit(new Callable<JsonArray>() {
    @Override
    public JsonArray call() throws Exception {
        return method2(a,b);
    }
});

/*** Stop the executor***/
if (f1.isDone() && f2.isDone()) {
    executorService1.shutdown();
}

// Consume
// Calling get() will block 
JsonArray f1 = homeFeedFuture.get();
JsonArray f2 = activityFeedFuture.get();

```
### Example 2: Using Java CompletableFuture

```java

final CompletableFuture<JsonArray> f1 = 
CompletableFuture.supplyAsync(() ->
                method1(a,b,c), executorService1);

final CompletableFuture<JsonArray> f2 = 
CompletableFuture.supplyAsync(() ->
                method2(a,b), executorService1);

List<JsonArray> list = new ArrayList<>();

// When the tasks are done, then consume them.
final CompletableFuture<Void> allcompleted =
        CompletableFuture.allOf(f1, f2);
allcompleted.thenRun(() -> {
            try {   
                    // This will not block 
                    list.add(f1.get());
                    list.add(f2.get());
                }
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
);
```
