# Workloads and Scheduling

Sometimes we need granularity in where we place our workloads as well as mitigating against certain failure scenarios by leveraging the capabilities containerisation gives us.

In this section we cover:

* Self-healing nature of `Deployments`
* Rolling updates and rollbacks
* Decoupling configuration from our container images via `secrets` and `configmaps`
* Leveraging `probes` to go beyond basic scheduling features to ensure our applications are running correctly.
* Express specific granularity and rules when it comes to scheduling decision.
