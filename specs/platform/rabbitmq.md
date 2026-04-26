---
id: rabbitmq
layer: platform
extends: []
---

# RabbitMQ

## Purpose

RabbitMQ is a high-feature AMQP/MQTT/STOMP broker whose default configuration is calibrated for a developer laptop, not a production cluster: the default `guest`/`guest` user can log in over the network on some images, classic queues lose data on broker restart unless the publisher and queue both opted into durability, `auto_ack` consumers acknowledge messages before processing them, a publisher with no `confirm_select` cannot tell whether the broker actually accepted a write, and a queue with no `x-max-length` happily eats the disk while another with no DLX silently drops rejections. Mirrored classic queues — the historical answer to "how do I replicate?" — were removed in RabbitMQ 4.0, so any team still reaching for them is shipping a broken topology. This spec pins the cluster topology, authentication, vhost and permission model, queue-type selection (quorum / streams / classic), reliability primitives (publisher confirms, manual acks, prefetch, DLX, length limits), and the operational baseline (memory/disk watermarks, metrics, declarative topology) so a RabbitMQ deployment behaves like a durable message bus rather than a fast-but-fragile in-memory queue.

## References

- **spec** `openobserve` — observability target for RabbitMQ broker logs and Prometheus metrics
- **external** `https://www.rabbitmq.com/` — RabbitMQ home
- **external** `https://www.rabbitmq.com/docs` — RabbitMQ documentation
- **external** `https://www.rabbitmq.com/docs/production-checklist` — Production deployment checklist
- **external** `https://www.rabbitmq.com/docs/quorum-queues` — Quorum queues
- **external** `https://www.rabbitmq.com/docs/streams` — Streams
- **external** `https://www.rabbitmq.com/docs/classic-queues` — Classic queues
- **external** `https://www.rabbitmq.com/docs/confirms` — Publisher confirms and consumer acknowledgments
- **external** `https://www.rabbitmq.com/docs/consumer-prefetch` — Consumer prefetch (QoS)
- **external** `https://www.rabbitmq.com/docs/dlx` — Dead-letter exchanges
- **external** `https://www.rabbitmq.com/docs/access-control` — Access control and permissions
- **external** `https://www.rabbitmq.com/docs/ssl` — TLS configuration
- **external** `https://www.rabbitmq.com/docs/prometheus` — Prometheus metrics plugin

## Rules

1. Pin the RabbitMQ server version via Docker image tag (`rabbitmq:<X.Y.Z>-management`) or signed package release; do not deploy from `latest`.
2. Run RabbitMQ as a cluster of three or more odd-numbered nodes (3, 5, 7) with the `pause_minority` partition-handling strategy and clocks synchronized via NTP; do not run production on a single-node deployment.
3. Synchronize the Erlang cookie across all cluster nodes with file permissions restricted to the RabbitMQ user; do not commit the cookie to source control.
4. Require TLS on client (5671), inter-node (25672), and management (15671) connections in non-local deployments; do not accept plaintext connections from outside `localhost`.
5. Restrict inter-node ports (4369, 25672, and the Erlang distribution port range) to cluster members at the firewall level; do not expose inter-node ports to the public network.
6. Delete the default `guest` user (or restrict it to `localhost`) and disable anonymous login (`anonymous_login_user = none`); do not run a production node with the default `guest`/`guest` credentials reachable.
7. Create a separate RabbitMQ user per application with its own credentials sourced from a secret manager and rotated on a documented schedule; do not share one set of credentials across applications.
8. Scope each application user with explicit `configure` / `write` / `read` regex permissions per vhost following least privilege; do not grant `.*` on all three permissions to non-administrative users.
9. Use separate virtual hosts per environment (`prod`, `staging`, `dev`) and per logical tenant; do not share a vhost across environments or tenants.
10. Declare exchanges, queues, bindings, policies, and Shovel / Federation links as committed configuration (definitions JSON, Terraform / RabbitMQ Operator); do not configure production topology interactively in the management UI.
11. Use quorum queues for durable, business-critical work (orders, payments, jobs whose loss would harm the business); do not use classic mirrored queues — they were removed in RabbitMQ 4.0.
12. Use streams for high-fanout, repeatable-read, or large-backlog workloads (e.g. backlogs above ~5M messages, replay required); do not use quorum queues as a stream substitute.
13. Use classic queues only for short-lived, transient, or per-connection work (e.g. RPC reply queues, exclusive queues); do not use classic queues for messages whose loss would harm the business.
14. Configure a dead-letter exchange (DLX), dead-letter routing key, and `x-delivery-limit` on every quorum queue; do not silently drop messages on rejection or repeated redelivery.
15. Set explicit per-queue resource limits (`x-max-length`, `x-max-length-bytes`, and `x-message-ttl` where relevant); do not run a production queue without bounded resource limits.
16. Use long-lived connections with multiple channels per process; do not open and close a connection per message and do not share a single channel across concurrent threads.
17. Use separate connections for publishers and consumers so consumer-side flow control does not throttle publishers.
18. Enable publisher confirms on every publishing channel and treat unacknowledged or `basic.nack` confirms as publish failures; do not assume `basic.publish` returning successfully means the broker has accepted the message.
19. Publish persistent (`delivery_mode = 2`) messages to durable queues for any payload that must survive a broker restart.
20. Consume with manual acknowledgments (`auto_ack = false`) and ack only after the handler has successfully completed; do not enable `auto_ack` for production consumers.
21. Set an explicit consumer prefetch (`basic.qos prefetch_count`) per channel sized for the consumer's processing rate; do not run a consumer without a prefetch limit.
22. Implement consumer handlers as idempotent operations; assume any message can be redelivered after a consumer crash or a `basic.nack` with `requeue = true`.
23. Configure the broker memory high-watermark in the documented safe range (0.4–0.7, default 0.6) and ensure the free-disk-space limit is at least the watermark size; do not run with the disk free-space limit below the memory threshold.
24. Enable the Prometheus metrics plugin (`rabbitmq_prometheus`), scrape its `/metrics` endpoint into the team's metrics store, and forward broker logs to the central log store; do not operate a production cluster without metrics scraping.
