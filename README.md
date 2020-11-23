![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DKHM RFC for Restore Command

![Markdownlint Action](https://github.com/DK-Hostmaster/DKHM-RFC-Restore/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/DKHM-RFC-Restore/workflows/Spellcheck%20Action/badge.svg)

2020-11-05
Revision: 1.0

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
  - [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
  - [XML and XSD Examples](#xml-and-xsd-examples)
- [Description](#description)
  - [Restore Domain During Grace Period](#restore)
- [XSD Definition](#xsd-definition)
- [References](#references)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This is a draft and proposal for changes to the process for domain name restoration via the DK Hostmaster EPP portal/service. The specification briefly touches on the registrar portal service, which mimicks the EPP service for consistency.

The overall [description of the concept][CONCEPT] of the registrar model offered by DK Hostmaster A/S provided as a general overview, where this RFC digs into the details of the cancellation/deletion of domain names in the context of an implementation proposal.

The first proposal was included in two of our RFCs:

- [DKHM RFC for Prepaid account information with the EPP Service][DKHMRFCPREPAID]
- [DKHM RFC for Delete Domain EPP Command][DKHMRFCDELETE]

This has now been extracted and has become this DKHM RFC.

<a id="about-this-document"></a>
### About this Document

We have adopted the term RFC (_Request For Comments_), due to the recognition in the term and concept, so this document is a process supporting document, aiming to serve the purpose of obtaining a common understanding of the proposed implementation and to foster discussion on the details of the implementation. The final specification will be lifted into the [DK Hostmaster EPP Service Specification][DKHMEPPSPEC] implementation and this document will be closed for comments and the document no longer be updated.

This document is not the authoritative source for business and policy rules and possible discrepancies between this an any authoritative sources are regarded as errors in this document. This document is aimed at the technical specification and possible implementation and is an interpretation of authoritative sources and can therefor be erroneous.

<a id="license"></a>
### License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

- 1.2 2020-11-23
  - Addition of additional links to resources
  - Correction to links pointing to redundant resources
  - Minor rephrasing and clarifications

- 1.0 2020-11-05
  - Initial revision

<a id="xml-and-xsd-examples"></a>
### XML and XSD Examples

All example XML files are available in the [DK Hostmaster EPP XSD repository][DKHMXSDSPEC].

The XSD changes are found in the extension specified in [RFC:3915], so the XSD definition is available in a separate file named [rgp-1.0.xsd][RGP1.0]. We have updated the version of our XSD, even though no changes are made to the DKHM XSD definition, but we want to communicate the inclusion and use of the external XSD definition.

The proposed extensions and XSD definitions are available in the version [4.1][DKHMXSD4.1] revision of the DK Hostmaster XSD, which is currently a draft and work in progress and marked as a  _pre-release_.

<a id="description"></a>
## Description

<a id="restore"></a>
### Restore Domain During Grace Period

As described in [RFC:3915][RFC3915], with a support for grace periods, it is possible to restore a domain name scheduled for deletion, (in the state `pendingDelete`).

DK Hostmaster will support the ability to restore for two use-cases:

1. Get a domain name back to the state active from a pending deletion specified by an explicit deletion request (delete command) or a automatic expiration
1. Get a domain name back to state active from a pending deletion, caused by missing financial settlement

Domain names might be suspended for other reasons, these will no be recoverable using the described restore facility, this will be indicated using the `serverUpdateProhibited` status.

Restoration has to take place during the redemption period and will not be possible after the grace period has expired.

The restoration is requested using the update domain command.

Based on feedback from our users, we have decided to not implement the `request` operation part of the restoration process, so you have to skip directly to the `report` part of the process.

This mean we will not support the state of  `pendingRestore` for the restore process.

The step to actually complete the restoration is the `report` operation, which look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0
     epp-1.0.xsd">
    <command>
        <update>
            <domain:update xmlns:domain="urn:ietf:params:xml:ns:domain-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:domain-1.0 domain-1.0.xsd">
                <domain:name>example.com</domain:name>
                <domain:chg/>
            </domain:update>
        </update>
        <extension>
            <rgp:update xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
                <rgp:restore op="report">
                    <rgp:report>
                        <rgp:preData></rgp:preData>
                        <rgp:postData></rgp:postData>
                        <rgp:delTime>2003-07-10T22:00:00.0Z</rgp:delTime>
                        <rgp:resTime>2003-07-20T22:00:00.0Z</rgp:resTime>
                        <rgp:resReason></rgp:resReason>
                        <rgp:statement></rgp:statement>
                    </rgp:report>
                </rgp:restore>
            </rgp:update>
        </extension>
        <clTRID>ABC-12345</clTRID>
    </command>
</epp>
```

Example is lifted from [RFC:3915].

The proposal is to the the report part act as an acknowledgement. The domain name is restored _as-is_ if possible, so the mandatory fields:

- `rgp:preData`
- `rgp:postData`
- `rgp:resReason`
- `rgp:statement`

Have to be specified, but values are ignored. As are the optional field, which however is optional and does not have to be specified:

- `rgp:other`

The mandatory fields:

- `rgp:delTime`
- `rgp:resTime`

Have to be specified and will be evaluated according to [RFC:3915][RFC3915].

- The `rgp:delTime` value has to match the deletion date and time.
- The `rgp:resTime` value will be ignored since, we do not handle the `request` operation.

As described in [RFC:3915], multiple report requests can be submitted, until success and within the allowed timeframe of possible restoration.

A response indicating unsuccessful restoration attempt will look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0">
    <response>
        <result code="2004">
            <msg>Delete date is incorrect</msg>
        </result>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example lifted from [RFC:5730] and modified.

A response indicating successful restoration attempt will look as follows:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<epp xmlns="urn:ietf:params:xml:ns:epp-1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:ietf:params:xml:ns:epp-1.0
     epp-1.0.xsd">
    <response>
        <result code="1000">
            <msg lang="en">Command completed successfully</msg>
        </result>
        <extension>
            <rgp:upData xmlns:rgp="urn:ietf:params:xml:ns:rgp-1.0" xsi:schemaLocation="urn:ietf:params:xml:ns:rgp-1.0 rgp-1.0.xsd">
                <rgp:rgpStatus s="pendingRestore"/>
            </rgp:upData>
        </extension>
        <trID>
            <clTRID>ABC-12345</clTRID>
            <svTRID>54321-XYZ</svTRID>
        </trID>
    </response>
</epp>
```

Example is lifted from [RFC:3915].

<a id="xsd-definition"></a>
## XSD Definition

The `restore` command is an extension to `update domain`. All is described in [RFC:3915]. The XSD has been included in our EPP XSD repository as `rgs-1.0.xsd` all lifted from [RFC:3915], please see [the repository][DKHMXSD4.1] for details.

The referenced XSD version is not deployed at this time and is only available in the [EPP XSD repository][DKHMXSDSPEC], it might be surpassed by a newer version upon deployment of the EPP service implementing the proposal, please refer to the revision of [EPP Service Specification][DKHMEPPSPEC] describing the implementation.

<a id="references"></a>
## References

1. ["New basis for collaboration between registrars and DK Hostmaster"][CONCEPT]
1. [DK Hostmaster EPP Service Specification][DKHMEPPSPEC]
1. [DK Hostmaster EPP Service XSD Repository][DKHMXSDSPEC]
1. [DKHM RFC for Prepaid account information with the EPP Service][DKHMRFCPREPAID]
1. [DKHM RFC for Delete Domain EPP Command][DKHMRFCDELETE]
1. [RFC:3915 "Domain Registry Grace Period Mapping for the Extensible Provisioning Protocol (EPP)"][RFC:3915]
1. [RFC:5730 "Extensible Provisioning Protocol (EPP)"][RFC:5730]
1. [RFC:5731 "Extensible Provisioning Protocol (EPP) Domain Name Mapping"][RFC:5731]
1. [RFC:8748 "Registry Fee Extension for the Extensible Provisioning Protocol (EPP)][RFC:8748]

[CONCEPT]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[DKHMEPPSPEC]: https://github.com/DK-Hostmaster/epp-service-specification
[DKHMXSDSPEC]: https://github.com/DK-Hostmaster/epp-xsd-files
[DKHMRFCPREPAID]: https://github.com/DK-Hostmaster/DKHM-RFC-Prepaid
[DKHMRFCDELETE]: https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain
[RFC:3915]: https://tools.ietf.org/html/rfc3915.html
[RFC:5730]: https://www.rfc-editor.org/rfc/rfc5730.html
[RFC:5731]: https://www.rfc-editor.org/rfc/rfc5731.html
[RFC:8748]: https://tools.ietf.org/html/rfc8748.html
[DKHMXSD4.1]: https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/dkhm-4.1.xsd
[RGP1.0]: https://github.com/DK-Hostmaster/epp-xsd-files/blob/master/rgp-1.0.xsd
