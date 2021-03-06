---
layout: page
permalink: /develop/
title: "Developer"
page-title: "Developer Resources"
breadcrumb: contribute
---

# Developing with the M-Lab Platform

M-Lab's mission of providing an open platform for Internet research depends on a diverse community that creates and deploys tools to collect measurement data. In order to achieve our objective of supporting global Internet research, we encourage the adoption of our resources for nearly all use cases, including within commercial environments. M-Lab’s primary concern is that clients and experiments comply with our privacy policies and do not interfere with the healthy functioning of the platform. This document is intended to provide guidance on expectations and best practices in the use of M-Lab.

## General Requirements

### Clients are required to provide informed consent about collection of personal information

M-Lab is committed to preserving user privacy, and the collection of data is governed under our [Acceptable Use Policy]({{ site.baseurl }}/aup) (AUP). According to the AUP, measurements must be actively initiated based on informed consent provided prior to installation of the client or initiation of the test. How consent is obtained is left to client developers, in consideration of their product and local privacy regulations. M-Lab requires that the notice provide a link to the privacy policies or terms and conditions that apply to the use of an experiment or client, including the [policies of the M-Lab platform]({{ site.baseurl }}/privacy).

M-Lab does not allow the storage of personally identifiable information (PII) on the platform. Experiments may collect the public-facing IP address, due to its necessity in understanding experiment results and in consideration of the minimal implication to users. If a client plans to collect any other personal information, it must be stored elsewhere. The collection of personal information in these instances must be clearly communicated to users and opt-in.

For more on M-Lab’s privacy requirements, please see our [AUP]({{ site.baseurl }}/aup).

## General Questions and Best Practices

### How to select an M-Lab server for testing

Client developers are welcome to use any production M-Lab site according to their preference and testing plans. However, developers should keep in mind that certain servers can encounter disruption or administrative maintenance with short to no notice. Planning should allow for changes in availability for servers. The easiest and recommended way to deal with changing server availability is to use our directory service.

We provide a directory service named [Locate](https://locate.measurementlab.net/admin/map/ipv4/all) (also historically referred to as mlab-ns) that provides structured data on the availability of servers on the platform. By default, the Locate service provides the client with the address of one of the servers geographically closest to the user. However, other information is available within the Locate service based on the parameters provided to the API, including a list of all available servers. Applications should not cache values returned by mlab-ns for a long time; we recommended that maintainers’ applications query the service in real time to ensure the availability of any server and to monitor for new servers on the platform. This is important whether or not a client bases its selection of geographic closeness. At a minimum, we recommend applications invalidate their cache and query the Locate service again when the connection fails to a server name cached from an old response.

For more information on using the Locate service, please refer to the [M-Lab NS Design Document](https://github.com/m-lab/mlab-ns/blob/master/DESIGN_DOC.md).

### Using the M-Lab Locate Service (mlab-ns) for Server Selection

M-Lab encourages developers to use the M-Lab Locate Service (mlab-ns) to route all client requests to the closest available M-Lab server. mlab-ns is a distributed service hosted on Google AppEngine that ‘routes’ HTTP requests for tests hosted on M-Lab to the closest available server, according to different policies. See the [M-Lab NS Design Document](https://github.com/m-lab/mlab-ns/blob/master/DESIGN_DOC.md) for the original list of requirements for mlab-ns.

The Locate service is enabled for the following M-Lab tests:

* NDT, NDT SSL
* Neubot DASH
* WeHe

In a typical scenario, an app or web-based NDT implementation will make a call to the locate service to determine which M-Lab server is closest and available to the user to conduct the test. Calls to the locate service should at a minimum include the tool name within the path, for example: https://locate.measurementlab.net/ndt

A response in JSON format is returned which the application then uses to provide the client a server to use to conduct the test:

```json
{"city": "Washington_DC", "url": "http://ndt.iupui.mlab1.iad05.measurement-lab.org:7123", "ip": ["4.35.238.203"], "fqdn": "ndt.iupui.mlab1.iad05.measurement-lab.org", "site": "iad05", "country": "US"}
```

When the request arrives to the locate service, the geolocation of the user is automatically included in the HTTP headers by the Google AppEngine infrastructure. The Locate service logs are not governed by M-Lab's open-data policy. Queries to the Locate service may be sent prior to acquiring user consent to run the actual test. Using the location from AppEngine, Locate service selects the nearest available M-Lab server and redirects the user to the corresponding URL. Additional query parameters are available, which are detailed in the [M-Lab NS Design Document](https://github.com/m-lab/mlab-ns/blob/master/DESIGN_DOC.md).

### Where does M-Lab place servers?

The M-Lab platform is designed to provide broadband users with measurements that reflect their Internet experience. To align with this mission, the M-Lab sites that users conduct tests against are typically located in networks that host Internet content or key infrastructure locations. These providers tend to be Internet exchanges, transit networks, or prominent hosting companies. The M-­Lab sites are typically physically collocated in the same building as major network providers’ routers.

Measurement Lab is an international platform that currently consists of over 125 sites around the world, a number that is always increasing. The majority of sites are located in North America and Europe. However, M-Lab seeks to build out a presence everywhere. As a result, M-Lab has server coverage in many key locations, including in prominent developing markets. For up to date information on the geographic reach, please see our [Server Map]({{ site.baseurl }}/status).

Building out an international presence that covers all users is a process that requires time and collaboration. We are always sensitive to the interests of partners and users, so we appreciate feedback on which regions it would be useful for M-Lab to be present in. We are also an open consortium that relies on the support of the Internet community. If a partner can connect us to potential hosts that meet our [Site Hosting Requirements](https://docs.google.com/forms/d/e/1FAIpQLScfO1gr0IFIXHuNMcbq7QgmmrN0nv3ucNXh3FzmJunwHn5Quw/viewform?formkey=dHNMZ2p0OU5TckxIUFg0RVNhSk5teEE6MQ#gid=0), we would appreciate the referral and can also contribute hardware resources toward establishing a site.

### Best practices on test scheduling and frequency

M-Lab welcomes automated and repeated testing. However, doing so requires user consent and care in implementation so that clients do not unintentionally overload the platform.

The M-Lab platform is well provisioned to handle a large amount of concurrent tests. However, in order to avoid potential site-reliability issues, **M-Lab imposes a rate limit of 40 tests per client per day**. For individual users running regular tests, 40 tests per day is more than sufficient. Clients that hit this rate limit will be provided a [204 No Content response](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#204){:target="_blank"}.

**For software or hardware integrations M-Lab recommends testing no more than 4 times per day.** Because client integrations are likely to have a large number of device installs, batch scheduled and interactive scheduled tests should be randomized over each day, to spread the distribution of their clients’ tests evenly. M-Lab recommends that client integrations use a [Poisson](https://en.wikipedia.org/wiki/Poisson_point_process){:target="_blank"} process to schedule no more than 4 tests per day at randomized times. For example, using a Poisson process to randomize test start times can be done by choosing each inter-test wait time from an exponential distribution. In python, the appropriate function for this is `random.expovariate`, but similar functions are available in all major programming languages.

## NDT

NDT is the test most commonly integrated into third party websites or applications. There is no API key needed to use the M-Lab infrastructure, though we do recommend that you make use of mlab-ns in developing your application. Developers may integrate M-Lab tests or tools into software applications or web sites, leveraging our server infrastructure and available tests to provide a service to site visitors or application users.

### Integrating NDT in JavaScript

NDT can be integrated quickly and easily using a basic iframe:
`<iframe src="//www.measurementlab.net/p/ndt-ws.html" align="middle"></iframe>`

For developers interested in a custom integration:

* The [current NDT JavaScript library](https://github.com/ndt-project/ndt/tree/master/HTML5-frontend)
* A custom integration in Angular.js is available as an [example](https://github.com/m-lab/mlab-speedtest)
* Work is in progress on an [official JavaScript client for the ndt7 protocol](https://github.com/m-lab/ndt-server/issues/237)

### Integrating NDT on Mobile or Desktop

For deployments of NDT outside of the browser, we provide several reference client repositories:

* Officially Supported
  * NDT 7 Go - [https://github.com/m-lab/ndt7-client-go](https://github.com/m-lab/ndt7-client-go)

* Community Supported Clients
  * NDT 7 Android - [https://github.com/m-lab/ndt7-client-android](https://github.com/m-lab/ndt7-client-android)
  * NDT 7 iOS - [https://github.com/m-lab/ndt7-client-ios](https://github.com/m-lab/ndt7-client-ios)

### Can I use M-Lab’s services without measurements being added to the public dataset?

No. Using M-Lab's test infrastructure and services requires the the resulting data to be shared. All data collected by M-Lab tests are available to the public without restriction under a [No Rights Reserved Creative Commons Zero Waiver](http://creativecommons.org/about/cc0).

Experiments typically record their measurement results based on the server’s observation of the test. These results are stored on the server until they are aggregated into the central dataset, and there is no mechanism to exempt certain tests from this collection. Please note that this is the extent of the data that is required to be published. If a client collects its own complementary information or performs additional testing, we do not require the disclosure of this information.

While M-Lab has an interest in the broader adoption of Internet measurement tools, the platform was created to provide public data. We commonly find that the concerns of commercial users or targeted deployments can be addressed through simple design choices and other considerations. However if these concerns prove to be insurmountable, the backend software used for experiments should be available as open source software and compatible with existing clients. Third-parties could run their own private measurement servers running the NDT server code that do not contribute to the M-Lab dataset.

### How can I protect commercially important information while using M-Lab’s NDT?

While experiments provide the option to include metadata about the system or client that has conducted the measurement, those fields are generally not required. Experiments such as NDT receive hundreds of thousands of measurements on a daily basis, the majority of which do not include any metadata about the client. In effect, this means that all that is recorded is that a device with an certain IP address conducted a test, without disclosing what software or hardware was used to perform the test or other circumstances. Given the increasing number of integrations that do not include this metadata, the measurement could be the result of any number of clients. If a potential partner would like to use NDT or other experiments without providing incidental information about its devices or user base, it can submit measurements without metadata and fall within that pool of users.

## Integration and Branding Questions

### Can commercial or closed source services use M-Lab?

Yes. Anyone may use M-Lab's services as long as they follow the [AUP]({{ site.baseurl }}/aup) and obtain user consent.

M-Lab’s policies require that experiment maintainers provide an example open source client that is compatible with the services hosted on the platform. However, the clients that use these server resources – whether developed by the original maintainer or independent third parties – are not required to be open source. These requirements were imposed in order to facilitate wide adoption of measurements and the platform’s public resources, beyond solely one researcher or client. For example, with the [Network Diagnostic Tool]({{ site.baseurl }}/tests/ndt), the majority of measurements come from independently-developed clients and integrations into closed source products.

While the specific licensing requirements of clients may differ, they are generally provided under licenses that allow for commercial and closed source distribution. The core M-Lab team has also sought to [contribute its code](https://github.com/m-lab) and research under permissive licenses in support of these objectives.

M-Lab welcomes commercial use cases, and would be happy to answer questions on how to optimally utilize our public resources.
