# Auditing Ansible Automation Platform: An Automation Challenge

## Introduction

Automation platforms sit in an unusual place in modern infrastructure. To do their job, they need access to everything: servers, network devices, cloud accounts, and secrets. That makes them one of the most valuable targets an attacker can reach. Compromise the system that holds the keys, and you often don’t need to compromise anything else.&#x20;

Red Hat Ansible Automation Platform (AAP) is a prime example. Organizations use it to patch fleets of servers, provision cloud resources, push network configurations, and enforce compliance at scale. To do that, it stores credentials for the systems it manages and holds the authority to run privileged code on them. A single misconfigured role, an over-privileged service account, or a poorly reviewed project can turn a convenience into an easy path to full environment compromise.&#x20;

Over the course of the 2-week research project, we set out to answer 3 main questions:

1. **How is AAP typically deployed and used**, and where do its trust boundaries lie?
2. **How can misconfigurations and design weaknesses in AAP be abused** by an attacker who gains a foothold?
3. **Can the auditing of these weaknesses be automated**, so that defenders can find them before attackers do?

This post will walk through what we've learnt from this project. We'll start with an overview of Ansible, then discuss about the main classes of weaknesses that we examined and how auditing can be automated. Finally, we'll share about the audit tool that we set out to develop for this project.&#x20;

## Overview of Ansible Automation Platform

### What is Ansible Automation Platform (AAP)?

[Ansible](https://docs.ansible.com/) is an automation engine that uses human-readable YAML _playbooks_ to describe the desired state of systems and the steps to reach it. On its own, Ansible is a command-line tool. [Ansible Automation Platform](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7) is the enterprise layer built around it, adding a web interface, API, access control, scheduling, credential management, auditing, and content distribution so that automation can be shared safely across teams.

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

AAP is the supported, commercial product from Red Hat. Its upstream open-source counterparts include [AWX](https://github.com/ansible/awx) (the basis for the automation controller) and the Ansible community project itself. If you’ve seen “Ansible Tower” mentioned in older documentation, that was the earlier name for what is now the automation controller. AAP is made up of the following core components, which are accessible via the web UI or RESTful APIs.&#x20;

<table><thead><tr><th width="215">Component</th><th width="488">Role</th></tr></thead><tbody><tr><td><strong>Automation controller</strong></td><td>A command-and-control center where playbooks are executed, job activity is monitored, and log outputs are centralized</td></tr><tr><td><strong>Execution environments (EE)</strong></td><td>Container images that bundle the Ansible execution engine and other modules that help users automate all aspects of IT environments and processes. </td></tr><tr><td><strong>Automation hub</strong></td><td>A repository for collections, can be online or private. A private hub support disconnected and on-premise setups by syncing collections and EE images from Red Hat's cloub hub</td></tr><tr><td><strong>Event-Driven Ansible (EDA)</strong></td><td>Listens for events from monitoring and security tools and triggers automation in response.</td></tr><tr><td><strong>Automation mesh</strong></td><td>An overlay network that distributes job execution across nodes, including remote sites and restricted networks.</td></tr><tr><td><strong>Platform gateway</strong></td><td>Handles authentication and authorization for the whole </td></tr><tr><td><strong>PostgreSQL</strong></td><td>Database backing the platform's data</td></tr></tbody></table>

### Core Concepts

When working with Ansible and AAP, several terminologies will come up repeatedly

* **Inventories** are collections of hosts and groups, and can be managed through a REST interface and synced from external sources
* **Projects** are collections/groups of playbooks, typically imported from a version control repository like Git
* **Credentials** are sensitive information such as SSH keys or passwords, and are attached to the processes that run the playbooks. AAP injects credentials into jobs, and some may be passed in via environmental variables
* **Job templates** define what to run, where and with what credentials. These help to standardize automation deployment and delegation, making deployments consistent and reusable
* **Workflows** chain multiple job templates together with conditional logic
* **Organizations, teams** and **roles** (RBAC) control who can see, edit or run each of these objects

Importantly, most of these objects are interlinked. A user with the permissions to run a job template can indirectly use the credentials attached to the project, even if they do not have the permissions to read them. A user who can edit a project may be able to change the code that the credentials execute. Although this mode of delegation of permissions is powerful and useful, it also causes many security-related decisions.&#x20;

### How AAP is used

A typical Ansible workflow looks like this:

1. The content (playbooks, roles, collections etc.) is written and stored in source control, on the local filesystem or pulled from the automation hub
2. A project syncs the new content into the controller
3. A job template ties a playbook to an inventory and the credentials needed for the job to run
4. Jobs can be launched in a myriad of ways, such as using schedules, workflows, via an automated script calling an API or in response to certain events. Jobs can also be launched manually via the web UI
5. Launched jobs are executed in an execution environment, and results or output are logged centrally.

Common use cases include system configuration, software deployment, patching and compliance environments. For example, Ansible can be used to automate installing Windows security patches on a regular schedule or to collect system information from an inventory.&#x20;

## Security Threats to AAP

The automation controller holds credentials, knows the inventory and can execute code against the hosts in it. An authenticated user with sufficient permissions to control it, or can read the content stored on the controller, can perform a wide range of attacks and escalate their privileges. Within AAP, there are several main classes of misconfigurations and weaknesses that can be exploited by attackers. [Syslifters](https://handbook.syslifters.com/pentesting-manual/ansible-automation-platform) has published a useful handbook for manual auditing of AAP instances

* **Authentication and authorization**: SSO misconfigurations, excessive session, over-privileged users or service accounts
* **Credentials exposure**: secrets exposed in playbooks, job outputs or extra variables
* **Supply chain**: who can publish to Automation Hub or Git, whether execution environments pull from trusted registries, collection signing and namespace ownership
* **Job execution**: ad-hoc command allow lists, privilege escalation via extra variables/Jinja, hybrid mesh nodes executing outside the managed perimeter
* **Information disclosure**: job stdout in the UI, verbose logging, backup and support bundle exports

Because of the level of trust that automation platforms tend to hold, they become attractive targets for attackers planning to get a foothold on a corporate network. Controllers contain an up-to-date inventory of managed systems, stored credentials for said systems, and the ability to execute code or system commands on these systems. Additionally, it is unrealistic to block access from a controller to managed systems as it would break IT operations, which makes it harder to restrict a compromised automation controller. &#x20;

## Automated auditing for AAP

### Why manual auditing falls short

Red Hat's published [hardening guides](https://www.redhat.com/en/blog/hardening-guidance-red-hat-ansible-automation-platform) should be treated as a natural baseline. It covers initial planning and architecture design, installation, initial configuration and Day 2 operations. However, there are several limitations to Red Hat's guides

* Following the recommendations does not guarantee security or compliance
* Security assessments must be performed in the context of the organization's own threats and constraints

Larger AAP installations may add many organizations, teams, credentials and templates, and these can change constantly. Manually auditing all of the objects in a large installation will take a very long time and can be prone to human error. A repeatable and automated approaching to auditing can help to reduce drift and make audit results comparable over time.&#x20;

### Data sources for automated auditing

Most of what is required for auditing AAP is available programmatically.&#x20;

* Gateway APIs are publicly accessible, and some expose the rest of the available API endpoints in an easily parseable format such as JSON.&#x20;
  * `/api/gateway/v1/`
  * `/api/controller/v2/`
  * `/api/galaxy/v3/`
  * `/api/galaxy/v3/swagger-ui/#/`
  * `/api/galaxy/v3/galaxy.json`
  * `/api/galaxy/pulp/api/v3/`
  * `/api/eda/v1/openapi.json`
* RBAC and token data is easily accessible through APIs, although they require authentication
* Stdout and other consolidated logs are also accessible through authenticated API endpoints, which can leak sensitive credentials or network information
* Some system configuration settings can be accessed unauthenticated via the API, exposing security flags and SSO configurations
* Package and version data can also be accessed unauthenticated via the API, allowing attackers to fingerprint the version of AAP in use and identify matching exploits

## Research Approach

### Goals and Scope

For this research project, we aimed to find out how much information an AAP deployment can reveal, and to whom. Our work tested which of the data sources described above yield information to an unauthenticated or authenticated attacker.&#x20;

* Target: AAP version 2.6, deployed as a containerized instance on a RHEL 9.6 VM. Additionally, several vulnerable and secure playbooks were deployed as historical and active jobs
* Scope: API enumeration and information disclosure. We did not attempt to gain code execution capabilities, alter system configurations or disrupt services

### Methodology and Findings

We started by proxying traffic through Burp Suite and exploring the platform on our own to get an understanding of what we were working with. We quickly learnt that there were several endpoints that describe the platform's own API - either by returning the complete API structure in JSON format, or even providing a Swagger interface for us to interact with the API. We then sought to figure out what APIs were available to unauthenticated users, or if credentials were required in order to interact with the application.&#x20;

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

To help with this test, we wrote a simple Python script to check for unauthenticated endpoints. Since endpoint listings were unauthenticated and machine-readable, we were able to pull entire lists of endpoints and then iterate through each one to retrieve its data. For each endpoint, we recorded whether it required authentication or were available anonymously. Most endpoints required some form of authentication, but a few could be accessed without logging in. Notably, some of these unauthenticated endpoints two kinds of useful information:

* **Version data**. Some unauthenticated endpoints revealed system and package versions. Combined with Red Hat's Security Data API, these identified the AAP release and mapped it to known CVEs. In our lab environment, we matched over 400 CVEs, only a fraction of which had publicly available fixes

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

* **Configuration data**. Other endpoints revealed system settings. This information included security configurations, authentication architecture and feature flags, which an attacker could use to plan targeted attacks on identity and trust paths.

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Authenticated endpoints were not spared either. Using valid administrator credentials, we reviewed what the API returned for stored objects. We paid special attention to job output and centralized logs, as the manual enumeration phase previously revealed that credentials could be exposed in the logs if the playbook was not properly secured. We were able to pull complete logs for historical and active jobs from the API, which could contain unredacted plaintext credentials if the proper settings were not set in the playbook (such as using `no_log`).

&#x20;

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption><p>An example of printed credentials being displayed in stdout</p></figcaption></figure>

## Automating our Audit

While each audit step in isolation was simple enough to perform by hand, it started to become very slow and tedious to check each API manually for misconfigurations. As the tests dragged on, fatigue and human oversight started making results inconsistent. To that end, AnsibleScout was developed as a solution to this issue.&#x20;

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

[AnsibleScout](https://github.com/Ryo-Soikutsu/AnsibleScout) automates the discovery, enumeration and results evaluation steps described earlier. It can discover API endpoints advertised by the platform deployment, perform authenticated and unauthenticated enumeration and produce a self-contained findings report.&#x20;

The generated report highlights the following information:

* Which endpoints required authentication and which were reachable without it
* Total findings, grouped by service or component
* Platform and component versions, their lifecycle status and any matched CVEs
* Potential credential- or session-like values detected in job outputs
* Matched security rules, such as weak password policies or missing security configurations

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

AnsibleScout was inspired by NCC Group's [ScoutSuite](https://github.com/nccgroup/scoutsuite) tool and was developed with AI assistance using Qwen4-Max. Given our time constraints, this workflow let us turn a slow, manual audit process into working code quickly. We then performed manual validation of the tool's behavior against our test environment, and confirmed that the returned findings matched responses that we obtained from manual testing.&#x20;

Despite our best efforts, there are several limitations with AnsibleScout. Credential scanning and vulnerability mapping rely on pattern matching and version matching, so some findings may be incorrect. The tool also only covers generic hardening guidelines and cannot account for an organization's own security context and use case. It is recommended to use this tool alongside human review to separate intended behavior from real exposure, and to double-check its output by hand.&#x20;

## Wrapping Up

Automation platforms like AAP sit in a powerful spot. They hold credentials, know your inventory, and can run code across your infrastructure. That's why they deserve the same level of scrutiny as any other crown jewel.&#x20;

Our research showed that a lot of useful information is available just by asking the application for it. Starting with nothing more than Burp Suite and some curiosity, we found that the API describes itself in full, and even gives you a neat user interface to interact with. Although this can be useful for developers and sysadmins, it also makes itself an easy target for attackers. A handful of these endpoints can be accessed anonymously, revealing configuration data and software versions, which can be matched against publicly available databases. Once authenticated, the available information become much more concerning. Job output and logs, both historical and active, can be retrieved with a simple request to an endpoint. A single misconfiguration in an old version of an executed playbook can leak sensitive credentials, which an attacker can use to escalate their privileges or to further their attacks.&#x20;

None of this required exploiting vulnerabilities in the platform itself. Most of it comes down to what is visible by default and how automation content is written. However, doing all of this by hand gets tedious fast, and tedious means inconsistent. AnsibleScout automates the enumeration, flags the things found to be risky, and packages the results into a single HTML report you can read and share. It’s a starting point and not a verdict. It can produce false positives, it only covers generic hardening guidance, and it can’t know your organization’s context, so treat its findings as leads to verify instead of hard facts.&#x20;
