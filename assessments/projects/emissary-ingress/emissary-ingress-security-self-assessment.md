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

Envoy Proxy serves as the foundational data layer for Kubernetes traffic control as well as administration in Emissary Ingress. Designed specifically to manage incoming traffic in Kubernetes settings. With its simple user interfaces and Kubernetes-native options, Emissary Ingress simplifies and facilitates the use of Envoy Proxy in Kubernetes installations. 			Advanced traffic routing capabilities are made possible by this open source project, which offers customisable routing based on hostnames, HTTP URLs, headers, and other parameters. To provide traffic splitting criteria, Emissary Ingress makes use of the Ingress resources provided by Kubernetes or custom annotations. These rules may be predicated on a number of variables, such as HTTP pathways, headers, or the proportion of traffic that is routed to distinct service versions.   This makes managing load and traffic control between multiple services easier. It also has the capabilities like SSL termination, rate limiting , TLS , load balancing, traffic splitting, authentication and routing, emissary-ingress serves as a control plane for managing traffic that arrives. It is designed to function effectively in a micro services architecture, enabling adaptable traffic control and routing settings.

Emissary Ingress allows for flexible traffic distribution by supporting dynamic routing depending on a variety of parameters, including HTTP pathways, headers, and hostnames. It also allows traffic to be switched between multiple service versions, making canary deployments, A/B testing, and incremental rollouts smoother and provides flexibility and scalability by enabling specific control over service routing and traffic flow. Because it can manage TLS certificates and handle SSL termination, it improves the security of incoming traffic and enables protected routing traffic.  Kubernetes's capabilities are improved by Emissary Ingress, which makes it simpler to handle incoming traffic, put routing plans into place, and improve security. In order to properly utilise Emissary-ingress, we usually want to install it within our Kubernetes cluster, set it up to handle incoming traffic based on the needs of our application, and then manage it using Kubernetes resources such as custom settings, annotations, or Ingress objects.


### Actors
These are the individual parts of your system that interact to provide the 
desired functionality.  Actors only need to be separate, if they are isolated
in some way.  For example, if a service has a database and a front-end API, but
if a vulnerability in either one would compromise the other, then the distinction
between the database and front-end is not relevant.

The means by which actors are isolated should also be described, as this is often
what prevents an attacker from moving laterally after a compromise.

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
The intended goals of the projects including the security guarantees the project
 is meant to provide (e.g., Flibble only allows parties with an authorization
key to change data it stores).

### Non-goals
Non-goals that a reasonable reader of the project’s literature could believe may
be in scope (e.g., Flibble does not intend to stop a party with a key from storing
an arbitrarily large amount of data, possibly incurring financial cost or overwhelming
 the servers)

## Self-Assessment Use

This self-assessment is created by the [project] team to perform an internal analysis of the
project's security.  It is not intended to provide a security audit of [project], or
function as an independent assessment or attestation of [project]'s security health.

This document serves to provide [project] users with an initial understanding of
[project]'s security, where to find existing security documentation, [project] plans for
security, and general overview of [project] security practices, both for development of
[project] as well as security of [project].

This document provides the CNCF TAG-Security with an initial understanding of [project]
to assist in a joint-assessment, necessary for projects under incubation.  Taken
together, this document and the joint-assessment serve as a cornerstone for if and when
[project] seeks graduation and is preparing for a security audit.

## Security Functions and Features

* Authentication: Emissary-ingress utilizes AuthService, a custom resource allowing users to provide an endpoint for authentication services. Once configured, each request hitting Emissary triggers Envoy to send a copy of that request to the service specified within an AuthService custom resource. This allows the service to inspect virtually any aspect of the request, including the header, protocol, and scheme. The service can then modify headers as it desires for request header sanitizing or the addition of them, further allowing requests to be routed or denied with a custom error response sent back to the request sender.
* Rate Limiting: The RateLimitService is a custom resource that also allows users to provide information about services that Emissary should reach out to for incoming requests regarding rate limiting decisions. Service implementations should be provided and will allow users to attach custom labels to requests, attach labels to request properties (client IP, hostnames, et cetera), and rate limit based on keys.
* TLS Support: A wide variety of configuration options for TLS is available to users and allows them to specify the minimum or maximum version of TLS allowed, require client certificates, specify a list of permitted cipher suites and ECDH curves, and provide a certificate revocation list.
* Availability: Built-in configuration options for CORS and circuit breaker configurations are available for use to help prevent service overloading. Configuring the maximum number of simultaneous connections, requests, pending requests, and retries prevent system abuse alongside IP allow and deny list configurations. For more specific use cases, several other miscellaneous options are available, including Lua scripts that run on each request. For example, with Lua, users commonly remove request headers before reaching AuthService and RatelimitService so that they can add custom headers that correspond to billing or authentication purposes, preventing end users from spoofing them.
* Filtering Malicious Traffic: Emissary maintains a publicly available Web Application Firewall configuration based on the OWASP core rule set. This satisfies PCI 6.6 compliance requirements and enables users to further configure their own custom security on the firewall.
* Observability: Access logs are placed on stdout for reading using kubectl logs. Format and local destination can be configured to the administrator’s wishes while using envoy_log_ settings. Information collected includes service, driver (HTTP or TLS), driver configuration, additional log headers, the maximum number of seconds to buffer access before sending logs to ALS, and a soft size limit for the access log buffer, among other options.
* Emissary-ingress puts the access logs on stdout for reading using kubectl logs. Local destination and log format can be configured to the administrator’s wishes using envoy_log_ settings, however these options only allow for logging locally to Emissary-ingress’ Pod. Information collected includes but is not limited to service, driver (HTTP or TLS access), driver configuration, additional log headers, the maximum number of seconds to buffer access before sending them to ALS, and a soft size limit for the access log buffer.


## Project compliance

* Compliance.  List any security standards or sub-sections the project is
  already documented as meeting (PCI-DSS, COBIT, ISO, GDPR, etc.).

## Secure development practices

* Development Pipeline.  A description of the testing and assessment processes that
  the software undergoes as it is developed and built. Be sure to include specific
information such as if contributors are required to sign commits, if any container
images immutable and signed, how many reviewers before merging, any automated checks for
vulnerabilities, etc.
* Communication Channels. Reference where you document how to reach your team or
  describe in corresponding section.
  * Internal. How do team members communicate with each other?
  * Inbound. How do users or prospective users communicate with the team?
  * Outbound. How do you communicate with your users? (e.g. flibble-announce@
    mailing list)
* Ecosystem. How does your software fit into the cloud native ecosystem?  (e.g.
  Flibber is integrated with both Flocker and Noodles which covers
virtualization for 80% of cloud users. So, our small number of "users" actually
represents very wide usage across the ecosystem since every virtual instance uses
Flibber encryption by default.)

## Security issue resolution

Related security flaws may be reported over the Slack channel or by email to the emissary ingress team. The group responds expeditiously.
In addition, users may visit https://status.getambassador.io/ where any problems that arise while the active products (Emissary, Edge Stack, Telepresence, Ambassador Cloud) are being fixed will be reported.

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
