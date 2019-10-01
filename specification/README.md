# RRDG Data Sharing Specifications

This document contains the **"Data Sharing" specification (Version 1)** created by the Registry-Registrar-Data-Group (RRDG, https://rrdg.centr.org). The specification was created by the "Data Sharing" sub-group, specifically by the following persons (alphabetical order):

  - Ashley La Bolle
  - Alex Mayrhofer
  - John Hollifield
  - Laurent Callens
  - Neal McPherson
  - Patrick Myles

The work makes use of the results of the "Registration Metrics" sub-group - the data format proposed is aligned to the definitions there. See https://rrdg.centr.org/projects/standards/registration-metrics/ for details.

# Overview of the Specification

The specification is split in two parts, named the **File Format** definition, and the **Transport/Infrastructure** definition. While the first section defines the representation and extent of the data to be transmitted, the second part defines the protocol details of acquiring these files.

The specification is primarily intended to by used by **Registries** as the producer and **Registrars** as the consumers of the information, however, the format **MAY** be used in other situations, such as information sharing between **Registrars** and their **Resellers**, or even individual domain name holders of larger portfolios. 

Requirements and the actual specifications were agreed amongst RRDG members.

# File Format Definition 

## Basic definitions

The file format used to transport Registration Metrics is **CSV**. The CSV **MUST** adhere to the following specifications for interopability:

  - The `,` character (COMMA, U+002C) is used as a field seperator
  - A header containing the column names is included as the first line (see below for column definitions)
  - Any time-related information is in UTC (Note: See "issues" for traditional reporting periods), and ist to be given in ISO 8601 format 
  - Numbers are represented using `.` (PERIOD, U+002E) as decimal delimiter, and no seperator characters. 

Files are named as `metrics-<tld>-<type>-<start>-<end>.csv` with:

  - `<tld>`: The A-Label string of the TLD in question, excluding the trailing dot. The A-Label string is chosen to avoid issues with character encoding in file names, and includes the `xn--` prefix.
  - `<type>`: One of `daily` or `monthly`
  - `<start>`: Start date of the reporting period, in `YYYYMMDD` format 
  - `<end>`: End date of the reporting period, in `YYYYMMDD` format 

Start and End dates of the reporting period are inclusive (eg. the file `metrics-mytld-daily-20190901-20190930` includes information for both Sept 01 and Sept 30). 

## Column definitions

The file **MUST** include the following columns in the given order. Columns for which no information is available **MUST** be empty, while a contents of `0` indicates a literal vlaue of 0.

| Column Name | Definition | type | RRDG Registration Metric definition |
|-----------:|:-----------|:-----|:------|
| `start`| Start date/time of the reporting period for the respective data row | datetime | |
| `end`| End date/time of the reporting period for the respective data row | datetime | |
|`registered` | Number of registered domains under management by the registrar at the end of the reporting period | Registered | 
| `delegated` | Number of delegated domains under management by the registrar, at the end of the report period | Delegated | 
| `registry_suspended` | Number of domains under management by registrar which are suspended at the end of the reporting period | Registry Suspended | 
| `pending_release` | Number of domains under management by registrar which are pending release | Pending Release | 
| `dnssec_enabled` | Number of domains under management with DNSSEC key material | DNSSEC Enabled | 
| `Locked` | Number of domains under management that have a lock product applied | Locked |
| `idn` | Number of IDN domains under management | Internationalised domain name (IDN) |
| `create` | Number of successful registrations in reporting period | Create | 
| `drop_catch` | Number of successful dropcatches in reporting period | Drop-Catch |
| `renew` | Number of successful renewals in reporting period | Renew |
| `expiry` | Number of domains expired in reporting period | Expiry |
| `delete` | Number of domains actively deleted | Delete | 
| `restore` | Number of domains recovered from pending release | Restore | 
| `release` | Number of domains under management becoming available again | Release |
| `registrar_transfer_in` | Number of domains successfully transferred to registrar during reporting period | n/a (one side of Registrar Transfer) |
| `registrar_transfer_out` | Number of domains successfully transferred away from registrar during reporting period | n/a (other side of Registrar Transfer) | 
| `registrant_transfer` | Number of domains successfully changed registrant | Registrant transfer | 

Note that for e.g. `registrar_transfer_in`, `restores` etc.: when the identical domain name is subject to several of such transactions in the reporting period, each seperate (successful) event is counted (rather than the unique domains names). 

## Example

TODO

# Transport Specification

The specified transport for the Registration Metrics files is HTTPS. Details of HTTP implementation are given below:

## URL structure

Each Registry (or other "publisher", eg. in the case of registrar-reseller communication) **MUST** define a base URI that is unique by TLD. The base URI **MUST** include a version number, and for this specification that version number **MUST* be set to `v1`. The recommended form of base URI is:

```
https://<host>/<optional path prefix>/v1/
```

For example `https://data.nic.tld/metrics/v1/`. The URI **MUST NOT** provide data over an unencrypted channel (aka HTTP), but the HTTP-version of the URI **MAY** redirect to the respective HTTPS URI.

## HTTP Methods

The service **MUST** support the `GET` method, and **SHOULD** support the `HEAD` method. 

## Request Parameters

The Service **MUST** support the following query parameters:

| Parameter name | Value description | required? | 
|--------------:|:-----------------|:-----------|
| `type` | `monthly` or `daily` to request the respective aggregation | no, default `daily`|
| `start` | Start of reporting period, in format `YYYY-MM-DD`, inclusive | no. Defaults to "12 months ago" for the monthly reports, and "60 days ago" for daily reports | 
| `end` | End of reporting period, in format `YYYY-MM-DD`, inclusive. | no. Defaults to "yesterday" |

Note that if more formats (besides CSV) are to be added in the future, an additional `format` parameter might be introduced. 

A server **MAY** limit the extent of the reporting periods, and **SHOULD** respond with a `400 Bad Request` response code in such cases. The body of the response **SHOULD** explain the problem.

## Content Type / Disposition

A client **SHOULD** include `application/csv` or `text/csv` content types in the `Accept:` header of the request.

The server **SHOULD** response with `application/csv` or `text/csv` content type headers, and **SHOULD** use the `attachment` value for the `Content-Disposition:` response header. The `filename` of that header **SHOULD** be set to the filename as described above.

## Authentication

The server **MUST** authenticate the client. The actual form of authentication depends on the relation between registry and registrar, and will likely include methods such as:

  - Basic Authentication
  - Digest Authentication
  - Bearer Token
  - JSON Web Token

Authentication via a request parameter is **NOT RECOMMENDED**, as that might expose the authentication credential in log files.

## Further specifications

The HTTP response code `418` **MUST** only be used for implementations complying to RFC 7168.

