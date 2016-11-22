# spring-boot-starter-akka
Based on spring-boot and akka integration<br>
1. The ActorSystem into spring's jurisdiction, can be injected automatically in the code Actorsystem<br>
2. Support automatic creation of Remote Actor<br>

how to use

* Add the following dependencies to the Spring Boot project's pom.xml:
```
 <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-akka</artifactId>
         <version>1.3.1.RELEASE</version>
 </dependency>
 ```
* Add the Akka Actor in the application.properties configuration as follows:
```
spring.akka.systemName=ClientActorSystem
spring.akka.config=client.conf
spring.akka.actorBeanClass=com.alibaba.akka.TestActor
spring.akka.actorName=ClientHandler
```
* Spring boot Starts and writes Remote Actor
```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.run(args);
    }

package com.alibaba.akka;
import com.alibaba.boot.akka.ActorBean;
import akka.actor.UntypedActor;
@ActorBean
public class TestActor extends UntypedActor {

    public void onReceive(Object arg0) throws Exception {
        System.out.println(arg0);

    }

}
```
Start the container, you can use the autoconfig function of spring-boot,
will automatically create ActorSystem and expose remote Actor (TestActor)
* How to connect to a remote Actor
```
        final ActorRef actor = system.actorOf(Props.create(PingLookupActor.class,
                                    "akka.tcp://ClientActorSystem@127.0.0.1:2552/user/ClientHandler"),
                                              "PingLookupActor");

        TimeUnit.SECONDS.sleep(5);
        for (int i = 0; i < 1000000; i++) {
            actor.tell(TaskProtos.Ping.newBuilder().setId(UUID.randomUUID().toString())
                        .setNow(System.currentTimeMillis()).build(),
                       ActorRef.noSender());
        }

    }

}


public class PingLookupActor extends UntypedActor {

    private final String path;
    private ActorRef     calculator = null;

    public PingLookupActor(String path){
        this.path = path;
        sendIdentifyRequest();
    }

    private void sendIdentifyRequest() {
        getContext().actorSelection(path).tell(new Identify(path), getSelf());
        getContext().system().scheduler().scheduleOnce(Duration.create(3, TimeUnit.SECONDS), getSelf(),
                                               ReceiveTimeout.getInstance(), getContext().dispatcher(),
                                               getSelf());
    }

    @Override
    public void onReceive(Object message) throws Exception {
        if (message instanceof ActorIdentity) {
            calculator = ((ActorIdentity) message).getRef();
            if (calculator == null) {
                System.out.println("Remote actor not available: " + path);
            } else {
                getContext().watch(calculator);
                getContext().become(active, true);
            }

        } else if (message instanceof ReceiveTimeout) {
            sendIdentifyRequest();
        } else {
            System.out.println("Not ready yet");

        }
    }

    Procedure<Object> active = message -> {
        if (message instanceof TaskProtos.Ping) {
            TaskProtos.Ping request = (TaskProtos.Ping) message;
            calculator.tell(message, getSelf());
        } else if (message instanceof TaskProtos.PingResponse) {
            TaskProtos.PingResponse result = (TaskProtos.PingResponse) message;
            System.out.println(result.toBuilder().toString());
        } else if (message instanceof Terminated) {
            System.out.println("Calculator terminated");
            sendIdentifyRequest();
            getContext().unbecome();
        } else if (message instanceof ReceiveTimeout) {

        } else {
            unhandled(message);
        }

    };
}

```
Spring-boot on the development of knowledge

<a href ="http://www.jianshu.com/users/aa6df7dd83ec/latest_articles">Spring-boot study notes</a>