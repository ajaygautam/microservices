# Best practices in microservices

The Holy Grail of software engineering is the ability achieve independence in all aspects of software engineering. Microservices achieves this by granularity of services at the application level (not just the source code level), and communicating using lightweight protocols.

# What is microservices

Microservices is an architectural approach that decomposes services into self-sufficient, independent, [context bounded](https://martinfowler.com/bliki/BoundedContext.html) modules. This definition is so similar to [SOA](https://en.wikipedia.org/wiki/Service-oriented_architecture)... that people are actually calling Microservices as: SOA done right.

# Drawbacks

> *The cost, pain and suffering that comes from microservices is hard to imagine until you start adopting it.*

The complexity of a monolithic application does not disappear when it gets re-implemented as a set of microservice applications. Some of the complexity gets translated into operational complexity. The runtime overhead and the operational complexity of microservices can overwhelm the benefits of the approach.

As number of services grow, it gets difficult to track the overall effects of an event, and visualize the system as a whole. Testing the system as a whole gets more complicated too.

Microservices are more prone to [distributed systems related failures](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing). Inter-service calls over a network have a higher cost in terms of network latency and message processing time than in-process calls within a monolithic service process.

# Benefits

**<u>Modularity</u>**: individual application are smaller, easier to understand, develop, test, and make more resilient. Because independent applications are small, they are also easier to replace with better versions of the application.

**<u>Scalability</u>**: microservices are implemented and deployed independently of each other, in independent processes. They can be monitored and scaled independently of all other services. Thus you need to scale only the components that truly need to be scaled.

**<u>Modernizing infrastructure</u>**: The microservices approach is considered as a viable mean for modernizing existing software application, where systems and teams are running into issues around scalability and slower-development speeds.

**<u>Distributed development</u>**: The microservices approach parallelize development by enabling small, autonomous teams to develop, deploy and scale their respective services independently. It also allows the architecture of an individual service to emerge through continuous refactoring.

# Best Practices

## Organization

> *[Conway's law](https://en.wikipedia.org/wiki/Conway%27s_law) states that software systems end up "shaped like" the organizational structure they are designed in or designed for.*

To adopt the microservices approach, organizations must setup new org structures to reflect the systems they are trying to build.

**<u>Small teams</u>**: If you need more than [two pizzas](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/two-pizza-teams.html) to feed a team, its time to split them into smaller teams. 

**<u>Autonomy</u>** Give teams ownership, autonomy and authority for the service(s) they are building.

## Architecture decisions

> *Engineering is all about trade-offs. Make them intentionally.*

**<u>API calls vs. Events/Streams</u>** for communication between services: API is synchronous - best for when a user is waiting. Downside: failure escalation. Events are asynchronous - best for a workflow style projects. Enables the Service Agent pattern.

**<u>Consistency matters</u>**: Establish structured logging, and consistent  naming: across services, across teams, across databases. This enables ease of moving people across teams, and scaling the organization.

Establish a culture of **<u>building performant services</u>**. Yes, it's cheaper to buy hardware compared to hiring engineers, however, this should be an excuse for writing sloppy code.

Establish a **<u>build-vs-buy</u>** policy. For most organizations... build service where you can add value, buy (reuse) everything else.

**<u>Multiple programming languages</u>** Yes... a microservices environment does enable an organization to support multiple programming languages. However, be aware of the costs. Few downsides to consider:
* Different languages have different tooling and licensing needs.
* People tend to form allegiances around programming languages.
* Cross-language context propagation is difficult
* Structured / consistent logging is difficult.

## Design Considerations

Adopt [Domain Driven Design](https://martinfowler.com/tags/domain%20driven%20design.html) to mark microservices boundaries.

RPC is slower than PC (in-memory) calls. Balance this trade-off while establishing service boundaries.

For Event based systems, events should be first class citizens, and should have their own data model. Best to use tools to generate code from event model definitions. This allows for generation of automated test code for the models as well.

Make state immutable. This enables immutable services that are easier to scale via parallel processing.

Add consistent transaction ids across all services to enable distributed tracing.

Standardize health checks, profiling and performance monitoring tools. Add them early on - perhaps as a template to all services.

Make security and compliance part of the design. Have independent services that work on this.

Add failure handling and resilience early on. Because services depend on each other, any failure will have cascading effect.
* Isolate the blast radius for any given failure by implementing circuit breakers - detect a problem and have a sensible fallback / default - return static content. (eg: hystrix)
* Build in retries and idempotency to handle inter-service errors.
* If a service fails times, have it take itself out of the pool, and notify monitoring system
* Implement dead-letter-queue for handling failures that can't be recovered from.
* Published failures to a monitoring system.

Implement a Reconciler service - a service that listens to events and ensures that all workflows are completed, events are processed, etc - specially for requests that span multiple services. (SAGA verifier? workflow checker? etc)

## Data isolation

[Database per service](https://microservices.io/patterns/data/database-per-service.html) Each service must have exclusive access to its own database. No one else (service or team) should be allowed to connect to it. All access to the data MUST be provided via APIs only.

For event based systems, a message stream should be owned by one service.

[SAGA](https://microservices.io/patterns/data/saga.html) is a more appropriate methodology to implement distributed transaction management compared to multi-phase-commit.

## Testing - code

The [test pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) still applies!

Added complexity around microservices bring new challenges for assuring the quality of software. Add Automated tests to verify inter-services interaction: eg: Spring cloud contract.

## Deployment

Implement DevOps processes to achieve full potential of a miscroservices environment. Automated deployment using well engineered pipelines.

Immutable infrastructure Immutable idempotent deployments: All deployments must be fully automated. No manual intervention.

Achieve High-Availability and BCP by deploying to multiple regions / zones.

## Monitoring

Operational visibility matters. If you can't see it - you can't improve it. Implement log aggregation systems (eg: Splunk) and Dashboards (eg: kibana).

Automate error detection and recovery/remediation/anatomy detection, and correct without human involvement. Same for logs - separate the signal from the noise.

## Infrastructure resiliency

Failures will happen. Plan for them.

Build a strategy for load testing. Run load tests against prod in off-peak times. Make sure services can identify load-test events vs. BAU events, and account for traffic accordingly.

Fault-injection - test framework to inject Failures in prod 

Automate destructive testing - prove that systems work even when services go down. Implement systems like [Chaos Monkey](https://netflix.github.io/chaosmonkey/) from the get go. Make it a policy. Run it in PROD - all the time.

# Anti Patterns

## Sharing data across services

Sharing of databases across services adds dependencies between teams when database changes are need. This will eventually lead to a [Distributed monolith](https://www.simplethread.com/youre-not-actually-building-microservices/).

## Logging too much

Logging is good, and developers tend to go overboard. In a microservices environment, logging is an infrastructure cost... and needs to be managed.
