# BidMart Current Architecture

In this document, we discuss the architecture of our project, BidMart.
We implement BidMart as a microservice-based marketplace platform. The system
separates concerns into multiple services so each business capability can evolve
independently while still participating in the same end-to-end auction and order
flow.

At the moment, the main containers in our system are:
- `bidmart-frontend` as the React client used by buyers and sellers.
- `bidmart-infrastructure` as the Spring Cloud Gateway entry point and Docker
  Compose deployment configuration.
- `bidmart-auth-service` for authentication, authorization, roles, sessions,
  OAuth login, and email verification.
- `bidmart-catalogue-service` for listings, categories, and catalogue search.
- `bidmart-auction-service-rust` for auction lifecycle, bidding, and auction
  closure.
- `bidmart-wallet-service-rust` for wallet balance, fund holds, top-up, and
  payment integration.
- `bidmart-order-and-notification-service` for order creation, order state, and
  user notifications.

Our current platform also depends on infrastructure services. Each business
service owns its own PostgreSQL database, Redis is used by the authentication
flow for session-related caching, and RabbitMQ is used for asynchronous events
between services such as auction updates, wallet provisioning, and order
creation. The entire local environment is deployed with Docker Compose.

## Current Context Diagram

In the context diagram, we show BidMart as a single system from the perspective
of its external users and third-party integrations. Buyers and sellers interact
with one marketplace platform, while the system itself communicates with Google
OAuth for identity verification, an email provider for verification messages,
and Midtrans for payment-related workflows.

![Current Context Diagram](assets/1.%20Current%20Context%20Diagram.png)

## Current Container Diagram

In the container diagram, we expand the system boundary and show how BidMart is
internally divided. The frontend sends requests to the API Gateway, and the
gateway routes them to the appropriate backend service. This design helps keep
authentication, catalogue management, auctions, wallet operations, and
order/notification concerns separated. It also makes integration clearer,
because service-to-service collaboration can happen through direct HTTP calls or
through events published to RabbitMQ.

In our current architecture, each service is responsible for its own data and
business rules. This reduces coupling at the code level, but it also means the
overall platform depends on coordination across multiple services and messaging
infrastructure.

![Current Container Diagram](assets/2.%20Current%20Container%20Diagram.png)

## Current Deployment Diagram

In the deployment diagram, we show how the system currently runs in development. The
frontend, gateway, backend services, PostgreSQL instances, RabbitMQ, and Redis
are deployed as separate containers inside one Docker Compose network. External
providers such as Google OAuth, SMTP, and Midtrans are outside the local
environment but are still part of the operational architecture because some
runtime flows depend on them.

This deployment style is practical for our local integration testing because it
keeps all major platform components available in one reproducible environment.
At the same time, it highlights our current architectural constraint that the
system is still strongly tied to a single-node local deployment model.

![Current Deployment Diagram](assets/3.%20Current%20Deployment%20Diagram.png)
