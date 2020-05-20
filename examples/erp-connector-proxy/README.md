# Baselining Business Process Automation across SAP and Microsoft Dynamics

##### Stefan Schmidt ([Unibright](https://unibright.io)), Kyle Thomas ([Provide](https://provide.services)), Daniel Norkin ([Envision Blockchain](http://envisionblockchain.com))
May 21, 2020

# Introduction

The "Baseline-SAP-Dynamics Demo" shows a setup of different Enterprise Resource Planning Systems ("ERPs") using the Baseline Protocol to establish a common frame of reference on the public Ethereum Mainnet. The demo extends the [Radish34 POC](https://docs.baseline-protocol.org/radish34/radish34-explained), showing a procurement process in a supply chain POC.

The open-source-available code of the development work on this demo evolved out of a Hackathon of the EEA Eminent Integration Taskforce members Unibright and Provide and is being made available alongside the Radish34 example.

# What is Baseline?

The Baseline Protocol is an approach to using the public Mainnet as a common frame of reference between systems, including traditional corporate systems of record, any kind of database or state machine, and even different blockchains or DLTs. It is particularly promising as a way to reduce capital expense and other overheads while increasing operational integrity when automating business processes across multiple companies.

The approach is designed to appeal to security and performance-minded technology officers.

You can find all the details on the Baseline Protocol [here](https://docs.baseline-protocol.org/baseline-protocol/protocol).

# Challenges and Scope of Work

The setting of tasks in the Community Bootstrapping Phase of Baseline [roadmap](https://docs.baseline-protocol.org/baseline-protocol/roadmap) include extraction of concepts out of the Radish34 demo case into the protocol level. This demo therefore wants to extend the Radish34 case by integrating off-chain systems of record, to work out major challenges and provide solutions to them. The learnings should be manifested in a reference implementation that can support standards on the protocol itself.

The Use-case shown in the demo follows this path:

* A buyer, using SAP ERP, creates a Request For Proposal and sends it out to 2 of his potential suppliers
* One supplier, using a Microsoft Dynamics D365 ERP, receives the Request For Proposal, turning it into a Proposal with different price tiers, and sending it back to the buyer

* The buyer receives this Proposal, runs a comparison logic between different received proposals (including those of other suppliers), decides for one specific proposal, creates a corresponding Purchase Order and sends this to the supplier

* The supplier receives the Purchase Order and continues the process

The shown use-case does not claim to be complete. For example, no Master Service Agreements are involved, and a productive process would continue with additional steps including Delivery Notes, Invoices and Payments, which not in the scope of this demo.

The participants discovered the following challenges to be addressed indispensably:

* Establishing a non-centralized rendezvous point for multiparty business process automation, with such place also providing a solution for automating the setup of a baseline environment for each process participant (here: a supplier or a buyer) on its own infrastructure (i.e., using the participant's own AWS or Microsoft Azure credentials); and

* Establishing a minimum domain model, abstracting from the baseline target objects and offering a process oriented entry point for systems of record to integrate; and

* Integrating systems of record via a suitable service interface.

The proposed architecture and solutions to these challenges are presented in the next sections.

# Architecture Proposal

The main idea is to orchestrate the container environment for each baseline participant in a way it supports the addressing of the mentioned challenges at best.

Baseline itself is a microservice architecture, where the different components of this architecture are residing in docker containers. The existing radish demo applies a UI on top of this architecture to play through the demo case.

The architecture proposal of this demo builds upon the existing microservices, and adds layers to extract communication and integration with baseline towards an external system.

![Microservice Containers](docs/images/image5.png)
Microservice container environment for a participant in a baselined business process.

**Baseline Containers**: The microservices providing the Baseline Protocol and Radish34 use-case, based on this [branch](https://github.com/ethereum-oasis/baseline/tree/init-core) in GitHub, including several key fixes (i.e., unwiring cyclic dependencies within the existing Radish34 environment) and enhancements (i.e., point-to-point messaging between parties, use of a generalized circuit for baselining agreements).

**Provide Containers**: Provide identity, key management, blockchain and messaging microservice API containers representing the technical entry point and translation layer for data and baseline protocol messages, and the provider of messaging infrastructure leveraged by the Baseline stack for secure point-to-point messaging.

**Unibright Proxy**: An extraction of the Unibright Connector (a blockchain integration platform), consisting of a simplified, context-related domain model and a RESTful api to integrate off-chain systems.

![](docs/images/image1.png)
The actual system of record is integrated by on premise or cloud based integration software in the domain of the respective Operator, leading to the "full stack."

![](docs/images/image6.png)
Each role in the process should run its own full-stack, connecting to the standardized Unibright Proxy by way of Shuttle.

# Challenge: Automating the setup of a baseline environment for each process participant

Implementing the demo use-case as described and demonstrated herein arguably illustrates levels of technical and operational complexity that would prevent most organizations from successfully applying the Baseline approach to their processes.

A viable rendezvous point where every participant in a multiparty business process can "meet in the middle" to ensure common agreements exists between each party (i.e., agreement on the use-case/solution) and each technical team (i.e., agreement on the protocols, data models, integration points, etc.) is a prerequisite to starting any actual technical integration. Such a rendezvous point can only be considered "viable" if it:

* is non-centralized; and
* can automate container orchestration across major infrastructure vendors (i.e., AWS and Microsoft Azure); and
* it can provide atomicity guarantees across all participants' container runtimes during protocol upgrades (i.e., to ensure forward compatibility and continuity for early adopters)

Shuttle is a bring-your-own-keys rendezvous point enabling turnkey container orchestration and integration across infrastructure vendors. Shuttle is a product built by Provide to de-risk multiparty enterprise "production experiments " using the Baseline Protocol, providing continuity to early adopters of the Baseline approach. Provide is actively contributing to the standards of the Baseline Protocol while commercially supporting Shuttle projects .

The following complexities related to enabling the Baseline Protocol for a multiparty process such as the one illustrated by the Radish34 use-case are addressed by Shuttle as an enterprise rendezvous point :

* Infrastructure
  * Containers (and the dependency graph)
  * Blockchain
    * HD Wallets / Accounts
    * Meta transaction relay (i.e., enterprise "gas pump")
    * Smart Contracts (i.e., deployment, interaction)
    * Organization Identity / PKI / x509
    * Key material (i.e., for advanced privacy, messaging, zkSNARKs
  * Baseline Protocol
    * Circuit Registry
    * Forward-compatibility
    * Point-to-point messaging (i.e., proof receipts, etc.)
    * Translation for DTO → Baseline Protocol

![](docs/images/image2.png)
Baseline smart contract deployment to Ropsten testnet -- as of today, new projects are automatically subsidized by the Provide platform faucet when transaction broadcasts fail due to insufficient funds on every testnet. This same meta transaction / relay functionality will be helpful to organizations who want to participate in mainnet-enabled business processes in the future but do not want to hold Ether (i.e., when the Baseline Protocol has been deployed to the public Ethereum mainnet).

![](docs/images/image15.png)
Baseline smart contract suite intricacy, as illustrated by the contract-internal CREATE opcodes issued from within the Shuttle deployer contract. This functionality will become a standardized part of the Baseline protocol.

![](docs/images/image16.png)
Container orchestration "work product" -- each organization, using its own AWS/Azure credentials, leverages Shuttle to automate the configuration and deployment of 13 microservice container runtimes to cloud infrastructure under their own auspices. Provide also has capability of supporting this for on-premise deployments via a rack appliance.

# Challenge: Establishing Domain Model and Proxy

As the Baseline Protocol itself is still in its bootstrapping phase, it was not possible to just use a perfectly working "Baseline" endpoint, and feeding it with perfectly designed and standardized data for the use case. To establish a development environment, in which all participants (e.g. distributed software teams) can continue working and are not blocking each other. One solution to this can be a proxy.

A proxy is an intermediate layer that you establish in an integration process. The proxy only consists of simple domain model descriptions and basic operations like "Get List of Objects", "Get Specific Object" or "Create new Specific object". So we created a Domain Model for the procurement use case we wanted to show, and designed basic DTOs ("Data Transfer Objects") for all the object types involved, like RequestForProposals, Proposals, PurchaseOrders and so on. We also generated a service interface for all these DTOs automatically, and an authentication service as well.

The proxy defines the entry point for all integration partners in the use case scenario, agreeing on a common domain model and service layer. Still, every participant runs its own proxy, keeping the decentralised structure in place.

The proxy does not perform any business logic on its own (apart from some basic example mappings to make the first setup easier), but is calling the Provide Shuttle stack in this demo using [this](https://github.com/provideservices/provide-dotnet) open source project which provides the bridge between the proxy and Baseline protocol by way of the Provide stack.

# Challenge: Integrating Systems of Record

## SAP ERP Integration

To help baselining SAP environments (following the Buyer role in this demo), Unibright configured the Unibright Connector (the integration platform of the Unibright Framework) to integrate and map the SAP models with the proxy automatically.

## Object Mapping in the Unibright Connector

![](docs/images/image20.jpg)
SAP Main Navigation Hierarchy for Purchasing Process, incl ME49 -> Price Comparison

![](docs/images/image19.jpg)
Request for Quotation for 2 materials

![](docs/images/image12.jpg)
Quotation to the Request, incl PriceScale referenced to the OrderItem

![](docs/images/image8.jpg)
Resulting Purchase Order for Supplier ("Vendor" 100073)

## Using the action Dashboard of the webversion of the Unibright connector to monitor SAP <> Proxy communication

## Microsoft Dynamics

To help baselining Dynamics 365 environments, Envision Blockchain built an extension called Radish34 for Dynamics 365 Finance and Operations. While this demo is showing the Dynamics 365 Supplier environment, it's important to note that the Radish34 extension is duly configured to support both roles (Buyer and Supplier). Below is a diagram showing the specific Dynamics 365 Finance and Operation modules used and the objects that are passing through the Radish34 extension.

![](docs/images/image11.gif)
Radish34 Implementation Flow Chart

After importing the extension, organizations will need to configure parameters to interact with the Unibright Proxy, setup Customer codes, Vendor codes, and setup custom Number sequences (which creates identifiers for Dynamics 365 objects).

![](docs/images/image3.png)
Radish34 Parameter Module

Supplier Role

When setting up Customers, you'll need to identify customers using the Customer Setup Module and input the External code used in the Radish Module. The Value is automatically filled out by the proxy.

![](docs/images/image14.png)
Customer Setup Module

You can use the Radish34 service operations feature to make periodic or on-demand calls of the UB Proxy and receive RFPs.

![](docs/images/image7.png)
Radish34 Service Operations Module

You can use the Sales Quotation module to view, adjust the prices for the items the Buyer is requesting, and send the quotation.

![](docs/images/image17.png)
Sales Quotation Module

The Radish 34 Outgoing Proposals module allows you to approve, and send the proposal to the Buyer

![](docs/images/image9.png)
Radish34 Outgoing Proposals Module

The Radish 34 Service Operations module will periodically check for incoming purchase orders from the Buyer

![](docs/images/image10.png)
Radish34 Service Operations Module (Unchecked for incoming purchase orders)

The Sales Order modules to look at the approved proposal from the Buyer and confirm the sales order

![](docs/images/image18.png)
Sales Order module