Based on spring-boot and akka integration
1. The ActorSystem into spring's jurisdiction, can be injected automatically in the code Actorsystem
2. Support automatic creation of Remote Actor

how to use

Add the following dependencies to the Spring Boot project's pom.xml:
 <Dependency>
         <GroupId> org.springframework.boot </ groupId>
         <ArtifactId> spring-boot-starter-akka </ artifactId>
         <Version> 1.3.1.RELEASE </ version>
 </ Dependency>
Add the Akka Actor in the application.properties configuration as follows:
Spring.akka.systemName = ClientActorSystem
Spring.akka.config = client.conf
Spring.akka.actorBeanClass = com.alibaba.akka.TestActor
Spring.akka.actorName = ClientHandler
Spring boot Starts and writes Remote Actor
@SpringBootApplication
Public class Application {

    Public static void main (String [] args) {
        SpringApplication application = new SpringApplication (Application.class);
        Application.run (args);
    }}


Package com.alibaba.akka;
Import com.alibaba.boot.akka.ActorBean;
Import akka.actor.UntypedActor;
@ActorBean
Public class TestActor extends UntypedActor {

    Public void onReceive (Object arg0) throws Exception {
        System.out.println (arg0);

    }}

}}
Start the container, you can use the autoconfig function of spring-boot, will automatically create ActorSystem and expose remote Actor (TestActor)
How to connect to a remote Actor
Public class PingClientSystemMain {

    Public static void main (String [] args) throws InterruptedException {

        Final ActorSystem system = ActorSystem.create ( "PingLookupSystem", ConfigFactory.load ( "pingRemoteLookup"));
        Final ActorRef actor = system.actorOf (Props.create (PingLookupActor.class,
                                                           "Akka.tcp: //ClientActorSystem@127.0.0.1: 2552 / user / ClientHandler"),
                                              "PingLookupActor");

        TimeUnit.SECONDS.sleep (5);
        For (int i = 0; i & lt; 1000000; i ++) {
            SetNow (System.currentTimeMillis ()). Build (), setUp (), setTimer ()
                       ActorRef.noSender ());
        }}

    }}

}}


Public class PingLookupActor extends UntypedActor {

    Private final String path;
    Private ActorRef calculator = null;

    Public PingLookupActor (String path) {
        This.path = path;
        SendIdentifyRequest ();
    }}

    Private void sendIdentifyRequest () {
        GetContext (). ActorSelection (path) .tell (new Identify (path), getSelf ());
        ScheduleOnce (Duration.create (3, TimeUnit.SECONDS), getSelf (), getContext (), system ()
                                                       ReceiveTimeout.getInstance (), getContext (). Dispatcher (),
                                                       GetSelf ());
    }}

    @Override
    Public void onReceive (Object message) throws Exception {
        If (message instanceof ActorIdentity) {
            Calculator = ((ActorIdentity) message) .getRef ();
            If (calculator == null) {
                System.out.println ( "Remote actor not available:" + path);
            } Else {
                GetContext (). Watch (calculator);
                GetContext (). Become (active, true);
            }}

        } Else if (message instanceof ReceiveTimeout) {
            SendIdentifyRequest ();
        } Else {
            System.out.println ( "Not ready yet");

        }}
    }}

    Procedure <Object> active = message -> {
        If (message instanceof TaskProtos.Ping) {
            TaskProtos.Ping request = (TaskProtos.Ping) message;
            Calculator.tell (message, getSelf ());
        } Else if (message instanceof TaskProtos.PingResponse) {
            TaskProtos.PingResponse result = (TaskProtos.PingResponse) message;
            System.out.println (result.toBuilder (). ToString ());
        } Else if (message instanceof Terminated) {
            System.out.println ( "Calculator terminated");
            SendIdentifyRequest ();
            GetContext (). Unbecome ();
        } Else if (message instanceof ReceiveTimeout) {

        } Else {
            Unhandled (message);
        }}

    };
}}

Spring-boot on the development of knowledge

Spring-boot study notes