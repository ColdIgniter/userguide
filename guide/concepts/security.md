# Security Guidelines

We take security seriously. ColdIgniter incorporates a number of features and techniques to either enforce good security practices, or to enable you to do so easily.

We respect the [Open Web Application Security Project (OWASP)](https://owasp.org) and follow their recommendations as much as possible.

The following comes from [OWASP Top Ten](https://owasp.org/www-project-top-10/) and [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/), identifying the top vulnerabilities for web applications and APIs. For each, we provide a brief description, the OWASP recommendations, and then the ColdIgniter provisions to address the problem. Only the features that exist in ColdIgniter are listed as provisions. A feature of CodeIgniter that is not ported is marked.

!!! note "Difference from CodeIgniter"
    CodeIgniter points to its Shield package for authentication and authorization, to the Throttler for rate limits, to `spark` commands and to PHPUnit. None of these exist in ColdIgniter. Authentication and authorization are written in your application, usually in a [filter](../incoming/filters.md).

## OWASP Top 10 2021

### A01:2021 Broken Access Control

Access control enforces policy such that users cannot act outside of their intended permissions. Failures typically lead to unauthorized information disclosure, modification, or destruction of all data or performing a business function outside the user's limits.

Common access control vulnerabilities include:

- Violation of the principle of least privilege or deny by default, where access should only be granted for particular capabilities, roles, or users, but is available to anyone.
- Bypassing access control checks by modifying the URL, internal application state, or the HTML page, or by using an attack tool modifying API requests.
- Permitting viewing or editing someone else's account, by providing its unique identifier (insecure direct object references).
- Accessing an API with missing access controls for POST, PUT and DELETE.
- Elevation of privilege: acting as a user without being logged in or acting as an admin when logged in as a user.
- Metadata manipulation, such as replaying or tampering with a token, a cookie or a hidden field to elevate privileges.
- CORS misconfiguration that allows API access from untrusted origins.
- Force browsing to authenticated pages as an unauthenticated user or to privileged pages as a standard user.

#### OWASP recommendations

Access control is only effective in trusted server-side code, where the attacker cannot modify the access control check or metadata.

- Except for public resources, deny by default.
- Implement access control mechanisms once and re-use them throughout the application, including minimizing Cross-Origin Resource Sharing (CORS) usage.
- Model access controls should enforce record ownership rather than accepting that the user can create, read, update, or delete any record.
- Unique application business limit requirements should be enforced by domain models.
- Disable web server directory listing and ensure file metadata and backup files are not present within web roots.
- Log access control failures, alert admins when appropriate.
- Rate limit API and controller access to minimize the harm from automated attack tooling.
- Stateful session identifiers should be invalidated on the server after logout.

#### ColdIgniter provisions

- [Public](structure.md#application-structure-public) folder, with application and system outside
- [Validation](../libraries/validation.md) library
- [Security](../libraries/security.md) library provides for CSRF protection
- [Sessions](../libraries/sessions.md) library
- [CORS](../libraries/cors.md) filter
- `log_message()` function for logging

!!! warning "Not ported yet"
    The Throttler for rate limits (`Throttle/Throttler.php` in CodeIgniter) is not ported.

### A02:2021 Cryptographic Failures

The first thing is to determine the protection needs of data in transit and at rest. For example, passwords, credit card numbers, health records, personal information, and business secrets require extra protection, mainly if that data falls under privacy laws or regulations. For all such data, ask:

- Is any data transmitted in clear text, in protocols such as HTTP, SMTP or FTP?
- Are any old or weak cryptographic algorithms or protocols used?
- Are default crypto keys in use, weak keys generated or re-used, or is key management or rotation missing? Are keys checked into source code repositories?
- Is encryption not enforced, for example are HTTP security headers missing?
- Is the server certificate and the trust chain properly validated?
- Are initialization vectors reused or not generated securely? Is an insecure mode such as ECB in use?
- Are passwords used as cryptographic keys without a key derivation function?
- Is randomness used for cryptographic purposes that was not designed for it?
- Are deprecated hash functions such as MD5 or SHA1 in use?

#### OWASP recommendations

- Classify data processed, stored, or transmitted by an application, and identify which data is sensitive.
- Don't store sensitive data unnecessarily. Data that is not retained cannot be stolen.
- Encrypt all sensitive data at rest.
- Ensure up-to-date and strong standard algorithms, protocols, and keys are in place; use proper key management.
- Encrypt all data in transit with secure protocols such as TLS. Enforce encryption with directives like HTTP Strict Transport Security (HSTS).
- Disable caching for responses that contain sensitive data.
- Do not use legacy protocols such as FTP and SMTP for transporting sensitive data.
- Store passwords using strong adaptive and salted hashing functions with a work factor, such as Argon2, scrypt, bcrypt or PBKDF2.
- Choose initialization vectors that fit the mode of operation, and never use one twice for a fixed key.
- Always use authenticated encryption instead of just encryption.
- Generate keys cryptographically randomly.
- Avoid deprecated cryptographic functions and padding schemes, such as MD5 and SHA1.
- Verify independently the effectiveness of configuration and settings.

#### ColdIgniter provisions

- The config for global secure access (`app.forceGlobalSecureRequests` in **app/Config/App.cfc**)
- `force_https()` function
- [Encryption](../libraries/encryption.md) library

!!! warning "Not ported yet"
    The `encrypt` setting of the database config is not used by the SQL Server connection.

### A03:2021 Injection

An application is vulnerable to attack when:

- User-supplied data is not validated, filtered, or sanitized by the application.
- Dynamic queries or non-parameterized calls without context-aware escaping are used directly in the interpreter.
- Hostile data is used within search parameters of an object-relational mapping to extract additional, sensitive records.
- Hostile data is directly used or concatenated. The SQL or command contains the structure and malicious data in dynamic queries, commands, or stored procedures.

Some of the more common injections are SQL, NoSQL, OS command, ORM, LDAP and expression language injection. The concept is identical among all interpreters. Source code review is the best method of detecting if applications are vulnerable to injections.

#### OWASP recommendations

Preventing injection requires keeping data separate from commands and queries:

- The preferred option is to use a safe API, which avoids using the interpreter entirely or provides a parameterized interface.
- Use positive server-side input validation. This is not a complete defense, as many applications require special characters.
- For any residual dynamic queries, escape special characters using the specific escape syntax for that interpreter. SQL structures such as table names and column names cannot be escaped, so user-supplied structure names are dangerous.
- Use LIMIT and other SQL controls within queries to prevent mass disclosure of records in case of SQL injection.

#### ColdIgniter provisions

- [URI security](../general/urls.md)
- [InvalidChars](../incoming/filters.md) filter
- [Validation](../libraries/validation.md) library
- `esc()` function, which escapes data for the html, js, url and attr contexts
- [IncomingRequest](../incoming/incomingrequest.md) class provides for input field filtering
- Support for [Content Security Policy](../outgoing/csp.md)
- [Query Builder](../database/query_builder.md)
- [Database escape methods](../database/queries.md)
- [Query bindings](../database/queries.md)

!!! warning "Not ported yet"
    The `css` context of `esc()` is not ported (`Common.php` of CodeIgniter).

### A04:2021 Insecure Design

Insecure design is a broad category representing different weaknesses, expressed as "missing or ineffective control design". There is a difference between insecure design and insecure implementation. A secure design can still have implementation defects. An insecure design cannot be fixed by a perfect implementation, as the needed security controls were never created to defend against specific attacks. One of the factors that contribute to insecure design is the lack of business risk profiling, and thus the failure to determine what level of security design is required.

#### OWASP recommendations

- Establish and use a secure development lifecycle with security professionals.
- Establish and use a library of secure design patterns or ready to use components.
- Use threat modeling for critical authentication, access control, business logic, and key flows.
- Integrate security language and controls into user stories.
- Integrate plausibility checks at each tier of your application.
- Write unit and integration tests to validate that all critical flows are resistant to the threat model.
- Segregate tier layers on the system and network layers depending on the exposure and protection needs.
- Segregate tenants robustly by design throughout all tiers.
- Limit resource consumption by user or service.

#### ColdIgniter provisions

- The framework gives you the building blocks of a secure design (filters, validation, the Security library), but ColdIgniter has no testing library of its own yet.

!!! warning "Not ported yet"
    The testing support of CodeIgniter (`Test/` in CodeIgniter) is mostly not ported, and the Throttler for rate limits is not ported.

### A05:2021 Security Misconfiguration

The application might be vulnerable if the application:

- Is missing appropriate security hardening across any part of the application stack.
- Has unnecessary features enabled or installed (ports, services, pages, accounts, or privileges).
- Has default accounts and their passwords still enabled and unchanged.
- Reveals stack traces or other overly informative error messages to users.
- Has the latest security features disabled or not configured securely on upgraded systems.
- Has security settings in the application servers, frameworks, libraries and databases that are not set to secure values.
- Does not send security headers or directives, or does not set them to secure values.
- Is out of date or vulnerable (see A06:2021).

Without a concerted, repeatable application security configuration process, systems are at a higher risk.

#### OWASP recommendations

- A repeatable hardening process makes it fast and easy to deploy another environment that is appropriately locked down. Development, QA, and production environments should all be configured identically, with different credentials in each.
- A minimal platform without any unnecessary features, components, documentation, and samples.
- A task to review and update the configurations as part of the patch management process.
- A segmented application architecture that provides effective and secure separation between components or tenants.
- Sending security directives to clients, for example security headers.
- An automated process to verify the effectiveness of the configurations and settings in all environments.

#### ColdIgniter provisions

- [Production mode](../general/environments.md) by default (the `CI_ENVIRONMENT` setting of the **.env** file)
- [SecureHeaders](../incoming/filters.md) filter

!!! warning "Not ported yet"
    The `spark config:check` command and the `spark phpini:check` command have no counterpart: ColdIgniter has no command line.

### A06:2021 Vulnerable and Outdated Components

You are likely vulnerable:

- If you do not know the versions of all components you use, including nested dependencies.
- If the software is vulnerable, unsupported, or out of date. This includes the operating system, web and application server, database, and all libraries.
- If you do not scan for vulnerabilities regularly and subscribe to security bulletins related to the components you use.
- If you do not fix or upgrade the underlying platform, frameworks, and dependencies in a risk-based, timely fashion.
- If software developers do not test the compatibility of updated, upgraded, or patched libraries.
- If you do not secure the components' configurations.

#### OWASP recommendations

There should be a patch management process in place to:

- Remove unused dependencies, unnecessary features, components, files, and documentation.
- Continuously inventory the versions of both client-side and server-side components and their dependencies, and monitor sources like the CVE and NVD lists for vulnerabilities.
- Only obtain components from official sources over secure links. Prefer signed packages.
- Monitor for libraries and components that are unmaintained or do not create security patches for older versions.

Every organization must ensure an ongoing plan for monitoring, triaging, and applying updates or configuration changes for the lifetime of the application.

#### ColdIgniter provisions

- See [Upgrading](../installation/upgrading.md) for how to move to a newer version of the framework. The framework has no dependencies that need a package manager.

!!! note "Difference from CodeIgniter"
    ColdIgniter is installed by copying files, so there is no Composer upgrade. ColdFusion MX 6.1 and Java 1.4.2 are old platforms that no longer receive security fixes, so keep the server away from untrusted networks.

### A07:2021 Identification and Authentication Failures

Confirmation of the user's identity, authentication, and session management is critical to protect against authentication-related attacks. There may be authentication weaknesses if the application:

- Permits automated attacks such as credential stuffing.
- Permits brute force or other automated attacks.
- Permits default, weak, or well-known passwords.
- Uses weak or ineffective credential recovery and forgot-password processes.
- Uses plain text, encrypted, or weakly hashed password data stores.
- Has missing or ineffective multi-factor authentication.
- Exposes the session identifier in the URL.
- Reuses the session identifier after successful login.
- Does not correctly invalidate session IDs during logout or a period of inactivity.

#### OWASP recommendations

- Where possible, implement multi-factor authentication.
- Do not ship or deploy with any default credentials, particularly for admin users.
- Implement weak password checks, such as testing new passwords against a list of the worst passwords.
- Align password length, complexity, and rotation policies with modern, evidence-based guidelines such as NIST 800-63b.
- Harden registration, credential recovery, and API pathways against account enumeration by using the same messages for all outcomes.
- Limit or increasingly delay failed login attempts, and log all failures.
- Use a server-side, secure, built-in session manager that generates a new random session ID with high entropy after login. The session identifier should not be in the URL, and should be invalidated after logout and timeouts.

#### ColdIgniter provisions

- [Session](../libraries/sessions.md) library

### A08:2021 Software and Data Integrity Failures

Software and data integrity failures relate to code and infrastructure that does not protect against integrity violations. An example is where an application relies upon plugins, libraries, or modules from untrusted sources, repositories, and content delivery networks. An insecure update process, or insecure deserialization of data that an attacker can modify, are further examples.

#### OWASP recommendations

- Use digital signatures or similar mechanisms to verify the software or data is from the expected source and has not been altered.
- Ensure libraries and dependencies are consuming trusted repositories.
- Ensure that a review process is in place for code and configuration changes.
- Ensure that your build and deployment process has proper segregation, configuration, and access control.
- Ensure that unsigned or unencrypted serialized data is not sent to untrusted clients without some form of integrity check.

#### ColdIgniter provisions

- n/a

### A09:2021 Security Logging and Monitoring Failures

This category helps detect, escalate, and respond to active breaches. Without logging and monitoring, breaches cannot be detected. Insufficient logging, detection, monitoring, and active response occurs any time:

- Auditable events, such as logins, failed logins, and high-value transactions, are not logged.
- Warnings and errors generate no, inadequate, or unclear log messages.
- Logs of applications and APIs are not monitored for suspicious activity.
- Logs are only stored locally.
- Appropriate alerting thresholds and response escalation processes are not in place or effective.
- Penetration testing and scans do not trigger alerts.
- The application cannot detect, escalate, or alert for active attacks in real time.

You are also vulnerable to information leakage by making logging and alerting events visible to a user or an attacker.

#### OWASP recommendations

- Ensure all login, access control, and server-side input validation failures can be logged with sufficient user context and held for enough time to allow delayed forensic analysis.
- Ensure that logs are generated in a format that log management solutions can easily consume.
- Ensure log data is encoded correctly to prevent injections or attacks on the logging or monitoring systems.
- Ensure high-value transactions have an audit trail with integrity controls to prevent tampering or deletion.
- Establish effective monitoring and alerting such that suspicious activities are detected and responded to quickly.
- Establish or adopt an incident response and recovery plan.

#### ColdIgniter provisions

- [Logging](../general/logging.md) library

### A10:2021 Server-Side Request Forgery (SSRF)

SSRF flaws occur whenever a web application is fetching a remote resource without validating the user-supplied URL. It allows an attacker to coerce the application to send a crafted request to an unexpected destination, even when protected by a firewall, VPN, or another type of network access control list.

#### OWASP recommendations

From the network layer:

- Segment remote resource access functionality in separate networks to reduce the impact of SSRF.
- Enforce "deny by default" firewall policies or network access control rules to block all but essential intranet traffic.

From the application layer:

- Sanitize and validate all client-supplied input data.
- Enforce the URL schema, port, and destination with a positive allow list.
- Do not send raw responses to clients.
- Disable HTTP redirections.
- Be aware of the URL consistency to avoid attacks such as DNS rebinding and time of check, time of use race conditions.

Do not mitigate SSRF via the use of a deny list or regular expression.

#### ColdIgniter provisions

- [Validation](../libraries/validation.md) library
- [IncomingRequest](../incoming/incomingrequest.md) class provides for input field filtering

## OWASP API Security Top 10 2023

### API1:2023 Broken Object Level Authorization

APIs tend to expose endpoints that handle object identifiers, creating a wide attack surface of object level access control issues. Object level authorization checks should be considered in every function that accesses a data source using an ID from the user.

#### OWASP recommendations

- Implement a proper authorization mechanism that relies on the user policies and hierarchy.
- Use the authorization mechanism to check if the logged-in user has access to perform the requested action on the record in every function that uses an input from the client to access a record in the database.
- Prefer the use of random and unpredictable values as GUIDs for the IDs of records.
- Write tests to evaluate the vulnerability of the authorization mechanism.

#### ColdIgniter provisions

- A [filter](../incoming/filters.md) or the code of your model can check the owner of a record. There is no ready made authorization library.

### API2:2023 Broken Authentication

Authentication mechanisms are often implemented incorrectly, allowing attackers to compromise authentication tokens or to exploit implementation flaws to assume other user's identities temporarily or permanently. Compromising a system's ability to identify the client or user compromises API security overall.

#### OWASP recommendations

- Make sure you know all the possible flows to authenticate to the API (mobile, web, deep links that implement one-click authentication and so on).
- Do not reinvent the wheel in authentication, token generation, or password storage. Use the standards.
- Credential recovery and forgot password endpoints should be treated as login endpoints in terms of brute force, rate limiting, and lockout protections.
- Require re-authentication for sensitive operations, such as changing the account owner email address or two-factor authentication settings.
- Implement anti brute force mechanisms to mitigate credential stuffing, dictionary attacks, and brute force attacks.
- Where possible, implement multi-factor authentication.
- Do not expose API keys in the URL.

#### ColdIgniter provisions

- [Filters](../incoming/filters.md)

!!! warning "Not ported yet"
    The `spark routes` command has no counterpart, and the Throttler for rate limits (`Throttle/Throttler.php`) is not ported.

### API3:2023 Broken Object Property Level Authorization

When authorizing a user to access an object through an API endpoint, it is important to validate that the user has permission to access the specific object properties they are trying to access. An API endpoint is vulnerable if it exposes properties of an object that are considered sensitive and should not be read by the user, or if it lets a user change, add or delete the value of a sensitive object property.

#### OWASP recommendations

- When exposing an object using an API endpoint, always make sure that the user should have access to the object's properties you expose.
- Avoid using generic methods such as to_json() and to_string(). Instead, cherry-pick specific object properties you specifically want to return.
- If possible, avoid using functions that automatically bind a client's input into code variables, internal objects, or object properties.
- Allow changes only to the object's properties that should be updated by the client.
- Implement a schema-based response validation mechanism as an extra layer of security.
- Keep returned data structures to a bare minimum, according to the business and functional requirements for the endpoint.

#### ColdIgniter provisions

- The `allowedFields` setting of a [model](../models/model.md)

### API4:2023 Unrestricted Resource Consumption

Satisfying API requests requires resources such as network bandwidth, CPU, memory, and storage. Other resources, such as emails, SMS, phone calls, or biometric validation, are made available by service providers via API integrations and paid for per request. Successful attacks can lead to denial of service or an increase of operational costs.

#### OWASP recommendations

- Use a solution that makes it easy to limit memory, CPU, number of restarts, file descriptors, and processes.
- Define and enforce a maximum size of data on all incoming parameters and payloads, such as a maximum length for strings, a maximum number of elements in arrays, and a maximum upload file size.
- Implement a limit on how often a client can interact with the API within a defined timeframe (rate limiting).
- Rate limiting should be fine tuned based on the business needs.
- Limit and throttle how many times or how often a single API client or user can perform a single operation.
- Add proper server-side validation for query string and request body parameters, specifically the one that controls the number of records to be returned in the response.
- Configure spending limits for all service providers and third-party integrations.

#### ColdIgniter provisions

- [Validation](../libraries/validation.md) library

!!! warning "Not ported yet"
    The Throttler for rate limits (`Throttle/Throttler.php`) is not ported.

### API5:2023 Broken Function Level Authorization

Complex access control policies with different hierarchies, groups, and roles, and an unclear separation between administrative and regular functions, tend to lead to authorization flaws. By exploiting these issues, attackers can gain access to the resources of other users or to administrative functions.

#### OWASP recommendations

- The enforcement mechanism(s) must deny all access by default, requiring explicit grants to specific roles for access to every function.
- Review your API endpoints against function level authorization flaws, keeping in mind the business logic of the application and the groups hierarchy.
- Make sure that all of your administrative controllers inherit from an administrative abstract controller that implements authorization checks based on the user's group or role.
- Make sure that administrative functions inside a regular controller implement authorization checks based on the user's group and role.

#### ColdIgniter provisions

- [Filters](../incoming/filters.md)

### API6:2023 Unrestricted Access to Sensitive Business Flows

APIs vulnerable to this risk expose a business flow, such as buying a ticket or posting a comment, without compensating for how the functionality could harm the business if used excessively in an automated manner. This does not necessarily come from implementation bugs.

#### OWASP recommendations

- Business: identify the business flows that might harm the business if they are used in an excessive manner.
- Engineering: choose the right protection mechanisms to mitigate the business risk, such as device fingerprinting, human detection, detecting non-human patterns, and blocking the IP addresses of Tor exit nodes and well-known proxies.
- Secure and limit access to APIs that are consumed directly by machines, such as developer and B2B APIs.

#### ColdIgniter provisions

- n/a

### API7:2023 Server Side Request Forgery

Server-Side Request Forgery (SSRF) flaws can occur when an API is fetching a remote resource without validating the user-supplied URI. This enables an attacker to coerce the application to send a crafted request to an unexpected destination, even when protected by a firewall or a VPN.

#### OWASP recommendations

- Isolate the resource fetching mechanism in your network.
- Whenever possible, use allow lists of remote origins users are expected to download resources from, URL schemes and ports, and accepted media types for a given functionality.
- Disable HTTP redirections.
- Use a well-tested and maintained URL parser to avoid issues caused by URL parsing inconsistencies.
- Validate and sanitize all client-supplied input data.
- Do not send raw responses to clients.

#### ColdIgniter provisions

- [Validation](../libraries/validation.md) library
- [IncomingRequest](../incoming/incomingrequest.md) class provides for input field filtering
- [URI](../libraries/uri.md) class

!!! warning "Not ported yet"
    The CURLRequest class (`HTTP/CURLRequest.php` in CodeIgniter) is not ported.

### API8:2023 Security Misconfiguration

APIs and the systems supporting them typically contain complex configurations, meant to make the APIs more customizable. Software and DevOps engineers can miss these configurations, or don't follow security best practices when it comes to configuration, opening the door for different types of attacks.

#### OWASP recommendations

- Establish a repeatable hardening and patching process.
- Have a process that oversees and continually assesses the configuration across the whole API stack.
- Automate a process to continually assess the effectiveness of the configuration and settings in all environments.
- Ensure that all API communications from the client to the API server and any downstream components happen over an encrypted communication channel (TLS).
- Be specific about which HTTP verbs each API can be accessed by: all other HTTP verbs should be disabled.
- Implement a proper Cross-Origin Resource Sharing (CORS) policy on APIs expected to be accessed from browser-based clients.
- Restrict incoming content types and data formats to those that meet the business or functional requirements.
- Ensure all servers in the HTTP server chain process incoming requests in a uniform manner.
- Where applicable, define and enforce all API response payload schemas, including error responses.

#### ColdIgniter provisions

- The config for global secure access (`app.forceGlobalSecureRequests`)
- `force_https()` function
- [Defined Route Routing](../incoming/routing.md)
- [CORS](../libraries/cors.md) filter

!!! warning "Not ported yet"
    Improved auto routing (`Router/AutoRouterImproved.php` in CodeIgniter) is not ported. Use defined routes.

### API9:2023 Improper Inventory Management

APIs tend to expose more endpoints than traditional web applications, making proper and updated documentation highly important. A proper inventory of hosts and deployed API versions also plays an important role in mitigating issues such as deprecated API versions and exposed debug endpoints.

#### OWASP recommendations

- Inventory all API hosts and document important aspects of each one of them, focusing on the API environment, who should have network access to the host, and the API version.
- Inventory integrated services and document important aspects such as their role in the system, what data is exchanged, and their sensitivity.
- Document all aspects of your API such as authentication, errors, redirects, rate limiting, cross-origin resource sharing (CORS) policy and endpoints, including their parameters, requests, and responses.
- Generate documentation automatically by adopting open standards.
- Use external protection measures such as API security specific solutions for all exposed versions of your APIs, not just for the current production version.
- Avoid using production data with non-production API deployments.

#### ColdIgniter provisions

!!! warning "Not ported yet"
    The `spark routes` command, which lists the routes, has no counterpart. The routes are listed in **app/Config/Routes.cfm**.

### API10:2023 Unsafe Consumption of APIs

Developers tend to trust data received from third-party APIs more than user input, and so tend to adopt weaker security standards. In order to compromise APIs, attackers go after integrated third-party services instead of trying to compromise the target API directly.

#### OWASP recommendations

- When evaluating service providers, assess their API security posture.
- Ensure all API interactions happen over a secure communication channel (TLS).
- Always validate and properly sanitize data received from integrated APIs before using it.
- Maintain an allow list of well-known locations integrated APIs may redirect yours to: do not blindly follow redirects.
- Limit resources available to process responses from third-party services.
- Implement timeouts for interactions with third-party services.

#### ColdIgniter provisions

- [Validation](../libraries/validation.md) library

!!! warning "Not ported yet"
    The CURLRequest class (`HTTP/CURLRequest.php` in CodeIgniter) is not ported.
