# Self-assessment
The Self-assessment is the initial document for projects to begin thinking about the
security of the project, determining gaps in their security, and preparing any security
documentation for their users. This document is ideal for projects currently in the
CNCF **sandbox** as well as projects that are looking to receive a joint assessment and
currently in CNCF **incubation**.

For a detailed guide with step-by-step discussion and examples, check out the free 
Express Learning course provided by Linux Foundation Training & Certification: 
[Security Assessments for Open Source Projects](https://training.linuxfoundation.org/express-learning/security-self-assessments-for-open-source-projects-lfel1005/).

# Self-assessment outline

## Table of contents

* [Metadata](#metadata)
  * [Security links](#security-links)
* [Overview](#overview)
  * [Actors](#actors)
  * [Actions](#actions)
  * [Background](#background)
  * [Goals](#goals)
  * [Non-goals](#non-goals)
* [Self-assessment use](#self-assessment-use)
* [Security functions and features](#security-functions-and-features)
* [Project compliance](#project-compliance)
* [Secure development practices](#secure-development-practices)
* [Security issue resolution](#security-issue-resolution)
* [Appendix](#appendix)

## Metadata

A table at the top for quick reference information, later used for indexing.

|   |  |
| -- | -- |
| Software | A link to the software’s repository.  |
| Security Provider | Yes or No. Is the primary function of the project to support the security of an integrating system?  |
| Languages | languages the project is written in |
| SBOM | Software bill of materials.  Link to the libraries, packages, versions used by the project, may also include direct dependencies. |
| | |

### Security links

Provide the list of links to existing security documentation for the project. You may
use the table below as an example:
| Doc | url |
| -- | -- |
| Security file | https://my.security.file |
| Default and optional configs | https://example.org/config |

## Overview

Formerly known as the Ambassador API Gateway,[Emissary-Ingress](https://www.getambassador.io)([emissary](https://github.com/emissary-ingress/emissary/tree/master )) project is an open-source API gateway that is native to Kubernetes that lets you control inbound traffic to your apps in a Kubernetes cluster. Ingress in Kubernetes refers to controlling external access to services that are operated within the cluster.

### Background

Understanding the importance and effects of Emissary Ingress in handling incoming traffic inside Kubernetes clusters would be made easier with some familiarity with the ideas surrounding Kubernetes (Ingress, Services, etc.). Within Kubernetes, Emissary Ingress functions as an Ingress controller. The Kubernetes cluster's ingress controllers manage incoming external traffic and route it to the relevant services.

[Envoy Proxy](https://www.envoyproxy.io) serves as the foundational data layer for Kubernetes traffic control as well as administration in Emissary Ingress. Designed specifically to manage incoming traffic in Kubernetes settings. With its simple user interfaces and Kubernetes-native options, Emissary Ingress simplifies and facilitates the use of Envoy Proxy in Kubernetes installations. 			Advanced traffic routing capabilities are made possible by this open source project, which offers customisable routing based on hostnames, HTTP URLs, headers, and other parameters. To provide traffic splitting criteria, Emissary Ingress makes use of the Ingress resources provided by Kubernetes or custom annotations. These rules may be predicated on a number of variables, such as HTTP pathways, headers, or the proportion of traffic that is routed to distinct service versions. This makes managing load and traffic control between multiple services easier. It also has the capabilities like SSL termination, [rate limiting](https://www.getambassador.io/docs/emissary/latest/topics/running/services/rate-limit-service), [TLS](https://www.getambassador.io/docs/emissary/latest/howtos/tls-termination), [load balancing](https://www.getambassador.io/docs/emissary/latest/topics/running/load-balancer), traffic splitting, [authentication](https://www.getambassador.io/docs/emissary/latest/topics/running/services/auth-service) and routing, emissary-ingress serves as a control plane for managing traffic that arrives. It is designed to function effectively in a micro services architecture, enabling adaptable traffic control and routing settings.

Emissary Ingress allows for flexible traffic distribution by supporting dynamic routing depending on a variety of parameters, including HTTP pathways, headers, and hostnames. It also allows traffic to be switched between multiple service versions, making canary deployments, A/B testing, and incremental rollouts smoother and provides flexibility and scalability by enabling specific control over service routing and traffic flow. Because it can manage TLS certificates and handle SSL termination, it improves the security of incoming traffic and enables protected routing traffic.  Kubernetes's capabilities are improved by Emissary Ingress, which makes it simpler to handle incoming traffic, put routing plans into place, and improve security. In order to properly utilise Emissary-ingress, we usually want to install it within our Kubernetes cluster, set it up to handle incoming traffic based on the needs of our application, and then manage it using Kubernetes resources such as custom settings, annotations, or Ingress objects.


### Actors

|   |     |
| -- | -- |
| Platform Adminstrators| Those who modify the configurations of the API gateway. Main focus on maintaining the Platform|
| Application Developers| Can Modify the Configuration of the API Gateway. Access only limited to Mapping and Routing Configurations|
| External Users| All the users whose request go through API Gateway|
|Apiext Server| Implements the Webhook Conversion interface for CRDs.|
|Diagd (Diagnostic Admin UI and Config Processor)| Provides a diagnostic admin UI, processes cluster changes into Envoy-ready configuration.|
|Ambex (gRPC Server for Envoy xDS)| Implements xDS APIs for dynamic Envoy configuration.|
|Envoy Proxy| Handles routing for all user traffic, dynamically updated using xDS services.|
|Ambassador Agent| Provides connectivity between the cluster and Ambassador Cloud.|
|Watch All The Things (Watt)| Watches for changes in the Kubernetes cluster, Consul, and the file system.|
|Entry Point (Entrypoint Binary and Busyambassador) | Manages the startup and coordination of various components in the container.|
| | |

### Actions

Emissary-ingress allows users and services to perform the following core actions:

Action      | Actors | Description     |
| :---       |    :----  |          :--- |
| Handling Ingress Traffic      |  <li>Emissary-ingress</li> <br> <li>Kubernetes API</li>       | Emissary checks the format of HTTP requests for protocol standard compliance, then validates the identity of incoming requests based on the preferred method of authentication (API tokens, certificates, et cetera) and RBAC. Rate limiting features limit service abuse. Envoy handles incoming and outgoing traffic amidst Kubernetes providing information about services and cluster management. Envoy handles rate limiting and TLS termination for abuse protection and data encryption/decryption for data passage to and from backend services.   |
| Request to Service Routing   | <li>Emissary-ingress</li> <br> <li>Envoy</li>        | Emissary routes incoming requests to its corresponding backend service based on rules and administrator configurations while Envoy executes routing decisions and forwards requests. Both actors validate paths against routing rules, validate headers for destination routing and injection prevention. Emissary also enforces any host-based routing rules.      |
Load Balancing | <li>Emissary-ingress</li> <br> <li>Envoy</li> | Emissary executes load balancing decisions based on rule configurations while Envoy implements the load balancing via request distribution across multiple instances of a service. Health checks of backend services are checked regularly and adjusted as necessary to avoid system freezing or downtime. Access to this service is restricted given the nature of the data handled (personally identifiable information [PII], backend service information, et cetera).
Monitoring and Logging | <li>Emissarry-ingress</li> <li> Envoy </li> </br> <li>Kubernetes API </li> </br> <li>Administrator</li> | Emissary-ingress will get its information from Kubernetes’ K8s cluster by watching for configuration settings within K8s resources, as well as Emissary’s CRD Resources. A consulWatcher is started if a user has configured a mapping to use ConsulRevolver. Admins utilize logging features to monitor and log activities that occur within Emissary and review them as needed for suspicious activities and/or incidents that require immediate attention and remediation.
Data Flow | <li>Watt</li> | Watt begins end-to-end data flow from developer configurations to Envoy configurations.

### Goals
Goal of emissary-ingress is to act as more native to kubernetes solution for a API Gateway, which supports functions like Layer 7 load balancer + Kubernetes Ingress built on Envoy Proxy. emissary achieves these functionalities leveraging envoy features and kubernetes features.

#### Security Goals
* Protect the declarative configurations such that no bad actor could modify them intern effect routing paths of the service.
  - Mitigation: Rbac
* Provide robust authentication features such that unpermitted accesses are allowed entry to the cluster
* Provide robust security around TLS features
* Provide security around service discovery
* Provide security against attacks like DDos using effective rate limiting.


### Non-goals
- emissary-ingress doesnt protect against risks such as Kubernetes API Server Bypass, which could have in a few number of ways with bad actors modifiying cluter configurations.
https://kubernetes.io/docs/concepts/security/api-server-bypass-risks/
- emissary-ingress doesnt protect against risks such as SQL injections, block malicious traffic, and monitor traffic in realtime. usually this done using a Web Application FireWall and can be integrated into API gateways to mitigrate such risks.
- 

## Self-Assessment Use

This self-assessment is created by the [emissary-ingress] team to perform an internal analysis of the
project's security.  It is not intended to provide a security audit of [emissary-ingress], or
function as an independent assessment or attestation of [emissary-ingress]'s security health.

This document serves to provide [emissary-ingress] users with an initial understanding of
[emissary-ingress]'s security, where to find existing security documentation, [emissary-ingress] plans for
security, and general overview of [emissary-ingress] security practices, both for development of
[emissary-ingress] as well as security of [emissary-ingress].

This document provides the CNCF TAG-Security with an initial understanding of [emissary-ingress]
to assist in a joint-assessment, necessary for projects under incubation.  Taken
together, this document and the joint-assessment serve as a cornerstone for if and when
[emissary-ingress] seeks graduation and is preparing for a security audit.

## Security Functions and Features

* **Authentication:** Emissary-ingress employs AuthService, a custom resource that enables users to choose an endpoint for authentication services. After being setup, every request that reaches Emissary causes Envoy to transmit a duplicate of that request to the designated service specified in an AuthService custom resource. This enables the service to examine almost every part of the request, including the header, protocol, and scheme. The service has the ability to edit headers according to its preferences for the purpose of sanitizing request headers or adding new ones. This allows for the routing or denial of requests, with the option to send a customized error response back to the sender of the request.

* **Rate Limiting:** The RateLimitService is a specialized resource that enables users to specify the services that Emissary should contact when making rate limiting choices for incoming requests. Users are required to offer service implementations that enable them to assign personalized labels to requests, associate labels with request parameters such as client IP and hostnames, and enforce rate limits based on keys. 

* **TLS Support:** Users have access to a diverse range of configuration options for TLS. These options enable users to define the minimum or maximum version of TLS that is authorized, mandate the usage of client certificates, provide a list of approved cipher suites and ECDH curves, and give a certificate revocation list. 

* **Availability:** There are pre-existing configuration options for CORS and circuit breaker settings that may be used to mitigate the risk of service overload. Setting the limits for the maximum number of concurrent connections, requests, pending requests, and retries helps to prevent system misuse, in addition to configuring IP allow and deny lists. Additional options are available for more particular use cases, such as Lua scripts that execute with each request. For instance, when using Lua, it is typical for users to eliminate request headers prior to reaching AuthService and RatelimitService. This allows them to include customized headers that are relevant to billing or authentication objectives, therefore preventing end users from falsifying them. 

* **Filtering Malicious Traffic:** Emissary maintains a publicly available Web Application Firewall configuration based on the OWASP core rule set. This satisfies PCI 6.6 compliance requirements and enables users to further configure their own custom security on the firewall.

* **Observability:** The access logs can be retrieved from the standard output (stdout) by using the command "kubectl logs".   The administrator may customize the format and local destination according to their preferences by using the envoy_log_ options.   The gathered information encompasses many data points such as the service type, driver type (HTTP or TLS), driver configuration, extra log headers, the maximum buffer time for access logs before sending them to ALS, and a flexible size limit for the access log buffer, among other configurable choices.
* Emissary-ingress puts the access logs on stdout for reading using kubectl logs. Local destination and log format can be configured to the administrator’s wishes using envoy_log_ settings, however these options only allow for logging locally to Emissary-ingress’ Pod. Information collected includes but is not limited to service, driver (HTTP or TLS access), driver configuration, additional log headers, the maximum number of seconds to buffer access before sending them to ALS, and a soft size limit for the access log buffer.


## Project compliance

* Compliance.  List any security standards or sub-sections the project is
  already documented as meeting (PCI-DSS, COBIT, ISO, GDPR, etc.).

## Secure development practices

Emissary-Ingress is in progress with 94% in Open Source Security Foundation (OpenSSF) best practices. [![OpenSSF Best Practices](https://www.bestpractices.dev/projects/1852/badge)](https://www.bestpractices.dev/projects/1852)

* Development Pipeline.  A description of the testing and assessment processes that
  All code is maintained in [GitHub](https://github.com/emissary-ingress/emissary) and changes must be reviewed by maintainers.
  - All the source code is publicly available on github
  - Development process is done through PRs on the master branch only, and Issue led.
  - Extensive documentation, resolution and targetted versions for the changes are required to be added to the Issue.
  - Every PR requires thorough testing, lint checks and corresponding Docs updation.
  - Commits need to be signed off and commit msgs are expected to be descriptive, include Issue links.
  - Each PR requires minimum 2 reviewer sign offs to be merged, and Maintainers will merge the PR.
  - All of the release branches are long-lived and have branch protection enabled, which will be used for security fixes or bug fixes.
  - Backport statergy: majority of the time patch branch will be based off from master and most Pull Requests will target master. ensuring bugs and fixes arent missed in the Next shipping version.
  - All PR requests trigger jobs that perform:
    - Unit Tests
    - E2E tests
    - lint
    - build
    - generate
    - check-envoy-protos
    - check-envoy-version
    - check-gotest
    - check-pytest
    - check-pytest-unit
    - check-chart
    - trivy-container-scan

* Communication Channels. Reference where you document how to reach your team or
  describe in corresponding section.
  * Internal. How do team members communicate with each other?
    Team members communicate with each other through the [Community Slack](https://a8r.io/slack), [Github issues](https://github.com/emissary-ingress/emissary/issues) or [Zoom meetings](https://ambassadorlabs.zoom.us/j/86139262248?pwd=bzZlcU96WjAxN2E1RFZFZXJXZ1FwQT09).
  * Inbound. How do users or prospective users communicate with the team?
    Users communicate with the team through the [Community Slack](https://a8r.io/slack), [Github issues](https://github.com/emissary-ingress/emissary/issues) or [Zoom troubleshooting meetings](https://us02web.zoom.us/j/83032365622).
  * Outbound. How do you communicate with your users? (e.g. flibble-announce@
    mailing list)
    Team members communicate with users through the [Community Slack](https://a8r.io/slack).

* Ecosystem. How does your software fit into the cloud native ecosystem?  (e.g.
  Flibber is integrated with both Flocker and Noodles which covers
virtualization for 80% of cloud users. So, our small number of "users" actually
represents very wide usage across the ecosystem since every virtual instance uses
Flibber encryption by default.)
 Emissary-ingress is a specialized control plane for Envoy Proxy. In this architecture, Emissary-ingress translates configuration (in the form of Kubernetes Custom Resources) to Envoy configuration. All actual traffic is directly handled by the high-performance Envoy Proxy. It can route traffic to kubernetes services and directly to the pods aswell as integrate with service meshes like consul, linkerd, isito.

## Security issue resolution

Related security flaws may be reported over the Slack channel or by email to the emissary ingress team. The group responds expeditiously.
In addition, users may visit [status.getambassador](https://status.getambassador.io/) where any problems that arise while the active products (Emissary, Edge Stack, Telepresence, Ambassador Cloud) are being fixed will be reported.

**Responsible Disclosures Process**

Assisting with the vulnerability fix, the emissary ingress works with The Common Vulnerability Scoring System (also known as CVSS Scores). In Detail, As soon as the team is able to put up the fix, a patched version will be made available for serious severity problems .The issues with Medium CVSS score will be fixed in the next version, which usually happens in 4-6 weeks or less. In a later version, they address Low 

**Incident Response**

As an API gateway that manages incoming traffic in Kubernetes clusters, Emissary Ingress concentrates on averting any security risks by taking a number of steps. After assigning the issue to a maintainer. The maintainer will investigate a known or existing CVE to determine the problem and the circumstances in which it may be exploited. They will next examine the Emissary/Edge Stack codebase to see if any of the packages listed in the CVE are being used. Or is this merely a matter of a scanner seeing that we have an earlier version of the dependency, or would they really call any of the methods particular to the CVE. If it is not an known vulnerability then maintainers Examine the source to see if there are any exploitable vulnerabilities that are calling functions that are a part of a susceptible package.

whether the problem is serious enough or not , Ambassador Labs and Engineering leadership will analyse it to see whether sending out emails or Slack notifications to users is a required outreach for high enough priority topics. Since Emissary is a distributor of Envoy Proxy, whenever a new CVE is discovered in Envoy, we have time to update Emissary to the most recent builds of Envoy and prepare a release for release as soon as the specifics of the security flaw are made public. 

## Appendix

* Known Issues Over Time. List or summarize statistics of past vulnerabilities
  with links. If none have been reported, provide data, if any, about your track
record in catching issues in code review or automated testing.
* [CII Best Practices](https://www.coreinfrastructure.org/programs/best-practices-program/).
  Best Practices. A brief discussion of where the project is at
  with respect to CII best practices and what it would need to
  achieve the badge.
* Case Studies. Provide context for reviewers by detailing 2-3 scenarios of
  real-world use cases.
* Related Projects / Vendors. Reflect on times prospective users have asked
  about the differences between your project and projectX. Reviewers will have
the same question.

# Emissary-Ingress Lightweight Threat Model
TAG-Security <2023-11-24>
<https://github.com/Rana-KV/tag-security/issues/3> 

## Overview

* Project: emissary-Ingress https://github.com/emissary-ingress/emissary/tree/master 
* Intended usage: open-source Kubernetes-native API Gateway
* Project data classification: Critical
* Highest risk impact: cluster breach, org takeover?
* Owner(s) and/or maintainer(s): 
  * Alex Gervais	alexgervais
  * Alice Wasko	aliceproxy
  * David Dymko	ddymko
  * Flynn		kflynn
  * Hamzah Qudsi	haq204
  * Lance Austin	lanceea
  * Luke Shumaker	lukeshu
  * Rafael Schloming	rhs
* Attendees and representation: 
  * <name (representation, email/GitHub contacts)>

## Threat Modeling Notes
The emissary-Ingress (emissary) project is an open-source API gateway that is native to Kubernetes and lets you control inbound traffic to your apps in a Kubernetes cluster. It is powered by an envoy proxy, and the emissary acts as a control plane which looks for changes to CRDs and updates the envoy proxy configurations in real-time. It is resilient and high performance comes from using Kubernetes and Envoy. Supports different types of protocols integrates well with service meshes and also provides features like authentication, TLS termination, rate limiting, and WAF integrations.

## Data Dictionary

|Name | Classification/Sensitivity | Comments|
| -- | -- | -- |
|Data type | High/Moderate/other | More info|
|CRDs | High | Kubernetes custom resources that define configs|
|Snapshots | Moderate | Diagnostics state outputs|
|Ir | Moderate | Intermediate envoy representations |
|RBAC Bindings | High | Authentication and authorization rules|
|Secrets |High |Security sensitive config like certs|
|Logs, Traces, Metrics | Low |  |


## Control Families

#### Deployment Architecture (pod and namespace configuration)
Emissary Ingress components are typically deployed into one namespace:
* emissary-system - Holds core infrastructure control plane pods like diagd, ambex etc. May also have an envoy ingress controller.

**Pods :**
  * emissary-ingress - Main Emissary pod with control plane and Envoy proxy.
  * emissary-apiext - Webhook conversion server pod.

**Services :**
  * emissary-ingress: Core Emissary ingress controller
  * emissary-apiext: Kubernetes conversion webhook

**Threats :**

  * Compromised deployment allows traffic inspection or routing manipulation

#### Networking (internal and external)

* Controls: pod networking and Kubernetes services for internal communication(internal). Envoy proxies handle all external ingress traffic and routing
* Data: Sensitive snapshot and configuration data shared between control plane components
* Threats:  A simple attack is accessing diagd/Ambex ports or interfaces meant only for Envoy. Could intercept sensitive external requests or route to malicious backends.

#### Cryptography

* Controls: all the network traffic outgoing from the emissary ingress is encrypted using TLS.
  * it uses certificates to authenticate clients requesting the services.
  * Data transfer from ingress to backend services in the cluster is encrypted using TLS.
  * The Container image of the Emissary ingress should be immutable.
  * The data stored in the Kubernetes storage ETDC should be Encrypted.
  * The communication to and from ETDC should be encrypted.
  * a web hook exposed from the APIEXT should verify the source when it gets a request.
* Data: The Initial configurations of the Emissary Ingress stored in ETDC.
  * The Snapshot of changes in the cluster is being transferred from WATT to Diadg via ETDC.
  * The CRD version snapshot is stored in ETDC and being transferred to Webhook.
  * The data signal and notifications are being sent from WATT and Diadg.
  * The Envoy config is transferred and persisted in ETDC
* Threat:
  * The Container image can be altered by the attacker in the storage or while being transferred.
  * The data stored in ETDC is not encrypted by default, Any attacker who has access to ETDC through other means can hamper the Emissary ingress data stored in ETDC.
  * The communication to and from ETDC is not encrypted, Attacker can intercept the traffic and hamper the data.
  * The Webhook exposed from APIEXT does not have any mechanism to verify the source of its request. This allows the attacker to change the CRD’s.
  * Attackers may attempt to hijack an established TLS session to gain unauthorized access.
  * If the certificate authority (CA) that issued the TLS certificate is compromised, attackers could create fraudulent certificates for the target domain.
  * Expired, revoked, or improperly configured certificates can lead to security vulnerabilities.
  * Outage of certificate services blocks ingress traffic

#### Multi-tenancy Isolation

* Controls: Relies on Kubernetes namespaces, RBAC, network policies for isolation. Tenant application traffic and access credentials are isolated. 
* Data: isolated namespaces for each tenant and their routing, filters, configuration data.
* Threats: Loss of RBAC controls risks tenant data leakage, compromised namespace could provide visibility across tenants.

#### Secrets Management

* Controls: 
  * N/A
* Data: 
  * Secrets management is not directly handled by the emissary ingress. It functions with kubernetes clusters which make use of Kubernetes secrets management features to handle sensitive data securely.
* Threats:
  * When hackers or unauthorized programs have access to the Kubernetes secrets which are used by the emissary ingress then it may result in the data breaches.

#### Authentication and Authorization

* Controls: 
  * The roles are assigned  by authorizations and the Kubernetes RBAC. which manages access controls for emissary-ingress.
  * Emissary could be configured by an AuthService to utilize a third party service for authorization and authentication. 
* Data:
  * Sensitive information like authentication tokens and certificates used by emissary may be kept safely as secrets in Kubernetes.
  * RBAC rules are established inside the cluster and interact with Kubernetes resources.
* Threats: 
  * Incorrectly set RBAC rules may result in providing inappropriate privileges.
  * Credential theft can gain access to the Kubernetes cluster

#### Storage

* Control: 
  * All data involved in the process are stored in Kubernetes Storage, ETDC.
  * Cache storage and temp storage is also used for emissary ingress functions.
  * Data stored in ETDC is shared among different components of Emissary ingress. Example: Snapshot stored by WATT is fetched by Diadg.
* Data:
  * Config changes applied to Clusters are stored in the form of Snapshots
  * ETDC stores snapshots of CRD.
  * Cache stores supporting data to generate Envoy Config from Intermediate representation..
  * Temporary storage is used to store the Intermediate representation.
  * Final Envoy Configs generated are stored in Kubernetes storage ETDC.
* Threat:
  * Since all the data is mainly stored in ETDC. It becomes a single point of failure. If by any chance data becomes faulty data will be lost permanently.
  * Attackers can gain administrative access to ETDC servers through other services running in the cluster. Will have complete control over the emissary ingress data as well.
  * If ETDC becomes unavailable, Emissary Ingress will not be able to operate.
  * If other services in the kubernetes cluster exhaust the storage in ETDC, Emissary Ingress data might get lost and Emissary ingress will not be able to operate properly.
  * Data stored in ETDC is not encrypted.

#### Audit and Logging

* Control:
  * Encryption of logs in transit and at rest via keys and certificates
  * RBAC services are utilized within Emissiary and Kubernetes to prevent unauthorized access. This type of access is required to get, list, watch, and update Ingress resources.
Client certificate validation via CA certificate can be enabled for extra security to validate clients against the server. This allows for client-side mTLS where both Emissary-ingress and the client provide and validate each other’s certificates.
Data:
HTTP access
Headers with and without sensitive data
Threats:
Tampering and deletion via disabling of log backup policies
Overwriting of log data
Threat Scenarios
We aim to identify threats at the lowest common denominator of deployment: consider the project at runtime, in a standard (non-hardened) deployment, on a major cloud provider.

Threats should be as broadly applicable as possible to consumers of the project.

Enumerate all approaches that an attacker may take to compromise systems. Once diffuse thinking is complete, double-check the working with STRIDE and the CIA triad.

This assessment considers threats in potential threat addresses. Unapplicable areas can be skipped and marked as “not applicable”. 

For each area, consider:

An External Attacker without access to the project at runtime or its hosting
An External Attacker with valid access to the project at runtime
An Internal Attacker with access to hosting environment (cluster or provider)
A Malicious Internal User

Theoretical Threats(STRIDE)
SPOOFING
Threat-01-S: An outside attacker could do IP address spoofing to bypass access controls and gain information about resources he should not.
Threat-02-S: An internal attacker with malicious intent could pose as admin, or user with elevated privileges and gain access to emissary configurations modifying traffic flow.
authenticating by mutual TLS could mitigate this by checking user identities.

TAMPERING
Threat-03-T: An attacker could gain access to Dynamic configuration used by envoy(aDS) and tamper with it leading to rerouting traffic and altered proxy behavior
Threat-04-T: An attacker who gained access could tamper with the emissary's dependencies such as envoy-proxy binaries and could affect the behaviour of the system adversely. 
Using tight integrity verification methods like checksums, signature verifications, audit logging and monitoring and security updates.

Repudiation 
Threat-05-R: An internal user could take advantage of a vulnerability and perform unauthorized actions such as changing the CRDs or modifying hosts, causing deviation in system behavior and denying any responsibility for it.
- Logging and audits need to be comprehensive and use immutable logging systems and verifiable signatures, making it very hard to tamper with it.

Information Disclosure
Threat-06-I: Information leak through metrics, logs.
Emissary-ingress metrics, and logs could have sensitive information, which could be leaked if inadequate controls are in place over access controls. Leading to data breaches.
Appropriate access controls in place, encryption for sensitive data and encryption for data in transit could act as mitigations.

Denial of Service
Threat-07-D: Causing system Outage
An internal attacker could find vulnerabilities in poorly configured RBAC and scale emissary pods down to 0, causing system outages.
Threat-08-D: Crashing control plane
An attacker could find vulnerabilities that can be leveraged to repeatedly crash the control plane components like ambex, and diag.
- Rate limiting, proxy redundancy, restrictive RBAC policies and availability zone spreads can help.

Elevation of Privilege
Threat-09-E: An attacker exploiting elevation of privilege in Emissary Ingress might gain unauthorized access to and modify various Kubernetes CRDs, affecting the overall security of the whole cluster. 
Deploy a robust tracking and log system to identify any illegal login attempts, abnormal activities, or changes in authorization that may suggest an increase in privileges. 


Potential Controls Summary
Mapping of threats to potential controls or remediations
Threat
Description
Controls
References
Deployment Architecture
Compromised deployment allows traffic inspection or routing manipulation or change in CRD definitions.
Pod and namespace configuration of Emissary-ingress components deployed into a namespace
strong access controls, monitoring for unexpected traffic patterns, regularly audit configurations for unauthorized changes. 


Networking
Unauthorized Diagd, Ambex ports, or interfaces meant only for Envoy for external request interception
The routing of traffic to malicious backend services Intercepting of sensitive external requests or routing requests to malicious backends
Emissary uses Pod networking and Kubernetes services for internal communication. The Envoy proxy handles all external ingress traffic and routing 


Storage
Storage as Single Point of failure
Since all the data is mainly stored in ETDC. If by any chance data becomes faulty data will be lost permanently.
Regularly backup etcd data to prevent permanent data loss. Use tools like etcdctl to automate the backup process.
https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/


Attack due to common resource
Attackers can gain administrative access to ETDC servers through other services running in the cluster. Will have complete control over the emissary ingress data as well.
Enable encryption for ETDC communication using Transport Layer Security (TLS). This ensures that data in transit is secure.

Implement client authentication for ETDC  to ensure that only authorized entities can access the ETDC cluster.


https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/


Resource Exhaustion attack
If other services in the kubernetes cluster exhaust the storage in ETDC, Emissary Ingress data might get lost and Emissary ingress will not be able to operate properly.
implement storage quotas to limit the amount of storage that can be consumed by individual services.
https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/


Cryptography
Image Alteration attack
The Container image can be altered by the attacker in the storage or while being transferred.


Use secure container registries, such as those supporting HTTPS, and consider image signing and verification mechanisms.


https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/


Attack on data at rest
The data stored in ETDC is not encrypted by default, Any attacker who has access to ETDC through other means can hamper the Emissary ingress data stored in ETDC.


To encrypt data stored in ETDC, use the --experimental-encryption-provider-config flag when starting the API server, specifying encryption providers.


https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
Attack on data in transit
The communication to and from ETDC is not encrypted, Attacker can intercept the traffic and hamper the data.
Enable encryption for etcd communication using Transport Layer Security (TLS). This ensures that data in transit is secure.
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/


Multi-Tenancy Isolation
Loss of RBAC controls risks tenant data leakage, and compromised namespace could provide visibility across tenants.






Repudiation
Role-Based Access Control
Emissary-ingress will need RBAC permissions to get, list, watch, and update Ingress resources
Enact strict RBAC services – which are already provided within Emissary and Kubernetes – to prevent unauthorized access. This type of access is required to get, list, watch, and update Ingress resources.
Emissary-ingress
Audit and Logging
Integrity
Tampering and deletion of logs via the disabling of log backup policies and/or the overwriting of log data
Entries should be stored in an immutable format such that data cannot be appended or deleted regardless of user privilege
Emissary-ingress


