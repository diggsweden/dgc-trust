# Document Signer Certificate (DSC) Trust List - Format Specification

> **Version:** 1.0.0

> **Date:** 2021-05-28

## Abstract

This document specifies the format for Document Signer Certificate (DSC) Trust List (DSC-TL).

The DSC-TL contains trusted DSC certificates for one or more countries representing Digital Green Certificate (DGC) Document Signers that are trusted according to the EU DGC Gateway.

## Terminology

Organisations adopting this specification for issuing health certificates are called `Document Signers` and organisations accepting health certificates as proof of health status are called `Verifiers`. Together, these are called `Participants`. `Document Signers` are registered under a country and `Verifiers` are assumed to be operating within a country infrastructure, relying on a national `Trust Point` for retrieving the information necessary to validate the authenticity of `Digital Green Certificates` (`DGC`) issued by `Document Signers` in other countries. The national `Trust Points` are operated by a competent appointed authority of that country. Countries operating together to form this infrastructure of trust are called `Member States`. Some aspects in this document must be coordinated between the `Member States`, such as the management of a namespace and the distribution of cryptographic keys. It is assumed that a function, hereafter referred to as the `DGC Gateway`, carries out these tasks.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 ([RFC2119](https://tools.ietf.org/html/rfc2119), [RFC8174](https://tools.ietf.org/html/rfc8174)) when, and only when, they appear in all capitals, as shown here.

## 1. Introduction

The EU Gateway provides an API that allows Trust Points in Member States to upload and download certificates used to support validation of Digital Green Certificates (DGC). Each Member State operates at least one Country Signing Certificate Authority (CSCA) that issues certificates for national Document Signers. These Document Signer Certificates are shared by the Trust Point with all national Verifiers.

This specification specifies the Swedish national infrastructure data format for distributing a list of trusted DSC, making up the DSC Trust List (DSC-TL).

## 2. DSC-TL Format

The DSC-TL is contained in a signed JSON Web Signature (JWS) according to [[RFC 7515](https://tools.ietf.org/html/rfc7515)].

The data is carried as Base64 payload according to [[RFC 7515](https://tools.ietf.org/html/rfc7515)].

### 2.1. JWS Header

The JWS header make use of the following header parameters:

Header Parameter | Value/Description | Precense
---|---|---
`typ`  | Set to the value `JOSE` to indicate that this is a JWS.|Mandatory
`alg` | Specifies the algorithm used to sign the JWS.|Mandatory
`x5c` | Carries the signing certificate and optionally any supporting certificate that may be used to validate the signing certificate.|Mandatory

### 2.2. JWS Payload

The JWS payload is a JSON object holding a value map of claims (similar to a JWT) according to the following table:

Claim | Value/Description | Precense
---|---|---
`iss`  | Issuer as defined in section 4.1.1 in [[RFC 7519](https://tools.ietf.org/html/rfc7519)]  |  Mandatory
`iat`  |  Issued at time as defined in section 4.1.6 in [[RFC 7519](https://tools.ietf.org/html/rfc7519)]  |  Mandatory
`exp`  | Expiration time as defined in section 4.1.4 in [[RFC 7519](https://tools.ietf.org/html/rfc7519)]  |  Mandatory
`dsc_trust_list`  | The DSC trust list (DSC-TL) holding a complete list of trusted DSC for each available country | Mandatory
`id`  | A unique identifier of an instance of the DSC-TL. If present, this ID MUST be updated everytime this DSC-TL is re-issued.|Optional
`aud`  | An array of string identifiers, each identifying an intended audience for this DSC-TL the value MUST be formatted in accordance with sectioin 4.1.3 of [[RFC 7519](https://tools.ietf.org/html/rfc7519)], but MUST be present as an Array of strings, even if only one audience is present | Optional

> NOTE: the only difference between the claims in this table and a JWT [[RFC 7519](https://tools.ietf.org/html/rfc7519)] is the use if the claim `id` instead of `jti`, and that the `aud` claim, if present, allways is an array instead of being a choice between an array or a string. The latter makes the schema and implementation easier and less ambiguous. Note also that because this is NOT a JWT, the header MUST declare the type value `JOSE`.

#### 2.2.1 DSC-TL

The DSC-TL provided in the claim `dsc_trust_list` holds a map where each country data is keyed under its 2 letter ISO 3166-1 alpha-2 country code. The object under each country holds 2 objects:

Object | Description
--- | ---
eku  | A map keyed under each DSC key ID (kid) expressing an array of Extended Key Usage OID (Object identifiers) defining the usage constraints for that DSC.
keys  |  The JWK [[RFC7517](https://tools.ietf.org/html/rfc7517)] key data for each DSC valid for the specified country.

#### 2.2.2 Design Considerations

The reason why the eku data is placed outside of the JWK set is to align with common implementations of JWK and to ensure the highest possible degree of compatibility with major software libraries. Even if the standard for JWK allows customized object items in the JWK, implementations such as Nimbus lacks the capability to natively add or parse such additional parameters.

Each DCS includes the EKU data, which can easily be extracted by standard tools. The "eku" is present as an optional resource for those implementations that would prefer not to parse any X.509 certificate ASN.1.

Keying material is provided as public key material as well as in the form of a complete DSC X.509 certificate. The certificate is considered the normative representation of the key as it also may contain metadata for the key such as the issuer name and usage constraining identifiers.

## 3. Schema

The following JSON schema defines the structure of the DSC-TL payload:

```
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "DSC-TL format JSON Schema",
    "description": "Schema defining the payload format for Document Signing Certificate - Trust List information",
    "type": "object",
    "required": [
        "iss",
        "iat",
        "exp",
        "dsc_trust_list"
    ],
    "properties": {
        "id": {
            "description": "Identifier of DSC-TL",
            "type": "string"
        },
        "iss": {
            "description": "Identifier of the issuer of the DSC-TL",
            "type": "string"
        },
        "iat": {
            "description": "Issued at time (seconds since epoch)",
            "type": "integer"
        },
        "exp": {
            "description": "Expiration time (seconds since epoch)",
            "type": "integer"
        },
        "aud": {
            "description": "Optional array of identifiers of the audiences",
            "type": "array",
            "items":{
                "description": "Identifier of an audience of this trust list",
                "type": "string"
            }
        },
        "dsc_trust_list": {
            "description": "List of trusted DSC for each country where the country code (ISO 3166-1 alpha-2) is the key",
            "type": "object",
            "patternProperties": {
                "^[A-Z]{2}$": {
                    "description": "Holds keys in the form of JWK Set and registered  EKU constraints for each registered DCS",
                    "type": "object",
                    "properties": {
                        "eku": {
                            "description": "Map of Extended Key Usage OID:s mapped by key ID (kid)",
                            "type": "object",
                            "properties": {
                                "patternProperties": {
                                    "^S{12}$": {
                                        "description": "Extended Key Usage OID as String",
                                        "type": "array",
                                        "items": {
                                            "type":"string"
                                        }
                                    }
                                }
                            }
                        },
                        "keys": {
                            "description": "JWK SET for all DSC for the specified country",
                            "type": "array",
                            "items": {
                                "description":"JWK object for a DSC",
                                "type": "object"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

## 4. Examples
The following examples illustrates a complete signed DSC-TL. 4.1 shows the complete JWS, 4.2 the decoded header and 4.3 the decoded payload in reader friendly format.

### 4.1 DSC-TL Example

```
eyJ4NWMiOlsiTUlJQllUQ0NBUWlnQXdJQkFnSUVYRzUzVFRBS0JnZ3Foa2pPUFFRREFqQTVNUXN3Q1FZRFZRUUdFd0pUUlRFWU1CWUdBMVVFQ2d3UFNVUnpaV01nVTI5c2RYUnBiMjV6TVJBd0RnWURWUVFEREFkRlF5QjBaWE4wTUI0WERURTVNREl5TVRFd01ESTFNMW9YRFRJd01ESXlNVEV3TURJMU0xb3dPVEVMTUFrR0ExVUVCaE1DVTBVeEdEQVdCZ05WQkFvTUQwbEVjMlZqSUZOdmJIVjBhVzl1Y3pFUU1BNEdBMVVFQXd3SFJVTWdkR1Z6ZERCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQlAwXC9pa2JhdnlWcGZPOCtUTzF4Q080RTlSMU91THlnWnJEeCsyTG1MNUJlQUE1WnlOTmZ5b2IwT0UrTDBcL2tGczZ4eVNHR1gyUTFJUm9CMktJb1ZTcDR3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnUkEzd3dScG9uamZOWE5RY0Q0QzUydndrWXY0NmZqc08zZlJUNENQV2NnMENJRm1KMTRPVFBzMDlqTUNDaUhOTElLekhxNVFieGxCUmZEUWRDXC9BRW9pelIiXSwidHlwIjoiSk9TRSIsImFsZyI6IkVTMjU2In0.eyJhdWQiOlsiZGdjLXZhbGlkYXRvcnMiXSwiZHNjX3RydXN0X2xpc3QiOnsiREUiOnsiZWt1Ijp7Imk0eSs5emUra2pFPSI6WyIxLjMuNi4xLjQuMS4wLjE4NDcuMjAyMS4xLjEiXSwiV1R2WCticjlcL1BjPSI6WyIxLjMuNi4xLjQuMS4wLjE4NDcuMjAyMS4xLjIiXX0sImtleXMiOlt7Imt0eSI6IkVDIiwieDV0I1MyNTYiOiJpNHktOXplLWtqRXFnZjU3SjFFSnRtLXAyMXVXcnVtQ1lZY3pwUjhPQ3dFIiwiY3J2IjoiUC0yNTYiLCJraWQiOiJpNHkrOXplK2tqRT0iLCJ4NWMiOlsiTUlJQllUQ0NBUWlnQXdJQkFnSUVYRzUzVFRBS0JnZ3Foa2pPUFFRREFqQTVNUXN3Q1FZRFZRUUdFd0pUUlRFWU1CWUdBMVVFQ2d3UFNVUnpaV01nVTI5c2RYUnBiMjV6TVJBd0RnWURWUVFEREFkRlF5QjBaWE4wTUI0WERURTVNREl5TVRFd01ESTFNMW9YRFRJd01ESXlNVEV3TURJMU0xb3dPVEVMTUFrR0ExVUVCaE1DVTBVeEdEQVdCZ05WQkFvTUQwbEVjMlZqSUZOdmJIVjBhVzl1Y3pFUU1BNEdBMVVFQXd3SFJVTWdkR1Z6ZERCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQlAwXC9pa2JhdnlWcGZPOCtUTzF4Q080RTlSMU91THlnWnJEeCsyTG1MNUJlQUE1WnlOTmZ5b2IwT0UrTDBcL2tGczZ4eVNHR1gyUTFJUm9CMktJb1ZTcDR3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnUkEzd3dScG9uamZOWE5RY0Q0QzUydndrWXY0NmZqc08zZlJUNENQV2NnMENJRm1KMTRPVFBzMDlqTUNDaUhOTElLekhxNVFieGxCUmZEUWRDXC9BRW9pelIiXSwieCI6Il9ULUtSdHFfSldsODd6NU03WEVJN2dUMUhVNjR2S0Jtc1BIN1l1WXZrRjQiLCJ5IjoiQUE1WnlOTmZ5b2IwT0UtTDBfa0ZzNnh5U0dHWDJRMUlSb0IyS0lvVlNwNCJ9LHsia3R5IjoiRUMiLCJ4NXQjUzI1NiI6ImlPcWdUZV9wdW44TGNCZjlyMG1kbXpHTE5jR2pxZnd3NGJ0WldWWnQ1dkkiLCJjcnYiOiJQLTUyMSIsImtpZCI6ImlPcWdUZVwvcHVuOD0iLCJ4NWMiOlsiTUlJQjZUQ0NBVXVnQXdJQkFnSUVYSEFYdURBS0JnZ3Foa2pPUFFRREFqQTVNUXN3Q1FZRFZRUUdFd0pUUlRFT01Bd0dBMVVFQ2d3RlNVUnpaV014R2pBWUJnTlZCQU1NRVU5d1pXNVRRVTFNSUVWRFF5QlVaWE4wTUI0WERURTVNREl5TWpFMU16a3pObG9YRFRJd01ESXlNakUxTXprek5sb3dPVEVMTUFrR0ExVUVCaE1DVTBVeERqQU1CZ05WQkFvTUJVbEVjMlZqTVJvd0dBWURWUVFEREJGUGNHVnVVMEZOVENCRlEwTWdWR1Z6ZERDQm16QVFCZ2NxaGtqT1BRSUJCZ1VyZ1FRQUl3T0JoZ0FFQVp3REFOVlNYUDVlTndPVjk4WjlhcXpOXC93SFpBVWk4YWp1YzBwU20wbElJNXZBTXBTRXZreWJUelNXRWRcL2RSRFB1UmJuRzFxd3VSeER6QklxV29jSEc2QUcwY2xkaExWQ2w0dlYzVDg5UFVBTDlkR1JiMTh1V253VFVPWWJ1OWM4Wnl1RTc5WU93ZmpJSnNxS0FcL1BCY2NwaTJEZzM1MTlvNlMySXl3eFdOSE5Qd0tNQW9HQ0NxR1NNNDlCQU1DQTRHTEFEQ0Jod0pDQU5jUXhtZVE0bjh6WTJscXJ0amhvOU1RS21iWXVPem9XejVKb1wvNGQrOU9PUlowVTlRMHo4RCtJRXRLVDRkZERmb1VMMGIwb0NHT1Y3TzB4YzNqekxsQU5Ba0U4azR2VjA4N2NiNFo2S1gyUXRORUhmMXFZb3lFeWI1UUtZbnU4a2pGa3ZGa2hRN1ZxM0dEUUYzZEdrTDI2RkVhU0wwZzZDdnBZR3piM2VcL2NxV296RjVnPT0iXSwieCI6IkFad0RBTlZTWFA1ZU53T1Y5OFo5YXF6Tl93SFpBVWk4YWp1YzBwU20wbElJNXZBTXBTRXZreWJUelNXRWRfZFJEUHVSYm5HMXF3dVJ4RHpCSXFXb2NIRzYiLCJ5IjoiQUcwY2xkaExWQ2w0dlYzVDg5UFVBTDlkR1JiMTh1V253VFVPWWJ1OWM4Wnl1RTc5WU93ZmpJSnNxS0FfUEJjY3BpMkRnMzUxOW82UzJJeXd4V05ITlB3SyJ9LHsia3R5IjoiUlNBIiwieDV0I1MyNTYiOiJXVHZYLWJyOV9QZndCam51eHV3VmQzUDZPdWlBYUxqcjJId05CakJGWUVjIiwiZSI6IkFRQUIiLCJ1c2UiOiJzaWciLCJraWQiOiJXVHZYK2JyOVwvUGM9IiwieDVjIjpbIk1JSUVKRENDQW95Z0F3SUJBZ0lFWG1KU3JEQU5CZ2txaGtpRzl3MEJBUXNGQURCR01Rc3dDUVlEVlFRR0V3SlRSVEVZTUJZR0ExVUVDZ3dQU1VSelpXTWdVMjlzZFhScGIyNXpNUjB3R3dZRFZRUUREQlJLYjJodUlFUnZaU0JVWlhOMElGTnBaMjVsY2pBZUZ3MHlNREF6TURZeE16TTVOVFphRncweU1UQXpNRFl4TXpNNU5UWmFNRVl4Q3pBSkJnTlZCQVlUQWxORk1SZ3dGZ1lEVlFRS0RBOUpSSE5sWXlCVGIyeDFkR2x2Ym5NeEhUQWJCZ05WQkFNTUZFcHZhRzRnUkc5bElGUmxjM1FnVTJsbmJtVnlNSUlCb2pBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVk4QU1JSUJpZ0tDQVlFQXNsREYyY3JPTFdDWTQ2Q2V0Y1V0STFQZHpjYzJWa0JKcGNySXdlb25qNGg0WjB0RlhPcmNqb1Urd2ZubGkzUDE5NVlKZnRHWVJxZ1FjSXd1OUZGOGF6NTg5QUg3UnlDODJJdm9iaG5oSExYd0RNVEhPZTd5a3BaMnRDXC9QV3E5UmcyMHJkRTRyRm9jblhcL1wvMUh5elo1Smp3VjFcL055SGJ1Y3g3dFVSSHBueHgrcU9sR243UjJlZnB6TFl2b2tGQVFTVktzenhhTWo0TytseDZMdDgzVUxIb1JaakR4VWxmOWtKYUtFTGVXQXNxZ0hsNFZicnk0MHRSWFNhZlB3TnZTaXJOY3h0OGRWOUl1UEkxM0x6K3pPS2dKbERaaWtYNStSUCtVcjc0TEpWOVBGOEY5WjdDUEVaQXVUb21TMU1FakdXNXg0WmdwajdVdk1aNzlLZ3hIT2wxVFprMEI5NEhvTWRQeUxINkhRNFM5Uzh5eTlPeGg0d3FxN3kzNndsUFRkNWRRNk5yU2hLR2hTck1tdzMxUk1HcVpSY2wwaW03ZzByc1VZbDVlUWNSVDF1S2d6RHAzbkVTZHhcLyswVmhWdjNFUm9EbTZKdXoyOE9MeStiSE50UFQ2RVFLU21uREp0NHN5WkU3QmFYcU02UWlCeWkxbGhETFRTUkVXRTVqbnRBZ01CQUFHakdqQVlNQWtHQTFVZEV3UUNNQUF3Q3dZRFZSMFBCQVFEQWdYZ01BMEdDU3FHU0liM0RRRUJDd1VBQTRJQmdRQ3I0bUk3VDRsXC9PNEVSMjBiZ283UWVcLzJPeVlrdjdCS0Z1R1VGWWVvRlhMcXNVM2h1TFB4c1FRVmpaK2lnSzFvNk9vSFdUNnBjb1EyRzZkejZBSmZnaEJlWlpPbDBORk1aOHd2XC81QUtiNHlNQW5OU1FNUzkrdmtcL2k4QU9RRDNSWHVLVlM0aTJ6QzBwQWtcL1VaVmszZzR3VnM4YzJaWWNSeXhBQ2xKTm9pSGlmampIdHVHU09FQlpKV3QyMkwzSU8rcGpjVHkzbFVBMzFjeFVvK0ozRitJWWFUVEVoZE5FaUhxSG1BWFZxMjgwQUpuUkplVFd2VnJrVXJpOVwvTnlnbytGeng5NXlheTVEdWF1NURmY2RRM2FyN1UxQ0wzNFBPMjRMOW1CR0FOUHZ5UEhXOGtqdlpWbUpvMEx5YlhJZFI1RlNHOHlhK1NPWG5DdkZVYmlTcm1yZm9IdWdjazhONG9CRDhNWDZwR1FJVVR1dVJBUm5wTVFzbTV0MWVGN2ZIK1NORFJoM1FxQjg5dkY3MUVyanJRVG01RkRPOFZ6M0huNjhZYWp4VGVrYWFmSGhyQzFHOUZFMnYxaUJyRU54NjhMZUY0QzBGSnBDRmxBVkRZS0tmaWtRbVNSTTRBUG14UXNLVytUNjhXQzRzM2tRbVQ0TFpFSStBOVwvRVNyYVhvRT0iXSwibiI6InNsREYyY3JPTFdDWTQ2Q2V0Y1V0STFQZHpjYzJWa0JKcGNySXdlb25qNGg0WjB0RlhPcmNqb1Utd2ZubGkzUDE5NVlKZnRHWVJxZ1FjSXd1OUZGOGF6NTg5QUg3UnlDODJJdm9iaG5oSExYd0RNVEhPZTd5a3BaMnRDX1BXcTlSZzIwcmRFNHJGb2NuWF9fMUh5elo1Smp3VjFfTnlIYnVjeDd0VVJIcG54eC1xT2xHbjdSMmVmcHpMWXZva0ZBUVNWS3N6eGFNajRPLWx4Nkx0ODNVTEhvUlpqRHhVbGY5a0phS0VMZVdBc3FnSGw0VmJyeTQwdFJYU2FmUHdOdlNpck5jeHQ4ZFY5SXVQSTEzTHotek9LZ0psRFppa1g1LVJQLVVyNzRMSlY5UEY4RjlaN0NQRVpBdVRvbVMxTUVqR1c1eDRaZ3BqN1V2TVo3OUtneEhPbDFUWmswQjk0SG9NZFB5TEg2SFE0UzlTOHl5OU94aDR3cXE3eTM2d2xQVGQ1ZFE2TnJTaEtHaFNyTW13MzFSTUdxWlJjbDBpbTdnMHJzVVlsNWVRY1JUMXVLZ3pEcDNuRVNkeF8tMFZoVnYzRVJvRG02SnV6MjhPTHktYkhOdFBUNkVRS1NtbkRKdDRzeVpFN0JhWHFNNlFpQnlpMWxoRExUU1JFV0U1am50In1dfSwiU0UiOnsiZWt1Ijp7Imk0eSs5emUra2pFPSI6WyIxLjMuNi4xLjQuMS4wLjE4NDcuMjAyMS4xLjIiLCIxLjMuNi4xLjQuMS4wLjE4NDcuMjAyMS4xLjMiXSwiaU9xZ1RlXC9wdW44PSI6WyIxLjMuNi4xLjQuMS4wLjE4NDcuMjAyMS4xLjEiXX0sImtleXMiOlt7Imt0eSI6IkVDIiwieDV0I1MyNTYiOiJpT3FnVGVfcHVuOExjQmY5cjBtZG16R0xOY0dqcWZ3dzRidFpXVlp0NXZJIiwiY3J2IjoiUC01MjEiLCJraWQiOiJpT3FnVGVcL3B1bjg9IiwieDVjIjpbIk1JSUI2VENDQVV1Z0F3SUJBZ0lFWEhBWHVEQUtCZ2dxaGtqT1BRUURBakE1TVFzd0NRWURWUVFHRXdKVFJURU9NQXdHQTFVRUNnd0ZTVVJ6WldNeEdqQVlCZ05WQkFNTUVVOXdaVzVUUVUxTUlFVkRReUJVWlhOME1CNFhEVEU1TURJeU1qRTFNemt6TmxvWERUSXdNREl5TWpFMU16a3pObG93T1RFTE1Ba0dBMVVFQmhNQ1UwVXhEakFNQmdOVkJBb01CVWxFYzJWak1Sb3dHQVlEVlFRRERCRlBjR1Z1VTBGTlRDQkZRME1nVkdWemREQ0JtekFRQmdjcWhrak9QUUlCQmdVcmdRUUFJd09CaGdBRUFad0RBTlZTWFA1ZU53T1Y5OFo5YXF6Tlwvd0haQVVpOGFqdWMwcFNtMGxJSTV2QU1wU0V2a3liVHpTV0VkXC9kUkRQdVJibkcxcXd1UnhEekJJcVdvY0hHNkFHMGNsZGhMVkNsNHZWM1Q4OVBVQUw5ZEdSYjE4dVdud1RVT1lidTljOFp5dUU3OVlPd2ZqSUpzcUtBXC9QQmNjcGkyRGczNTE5bzZTMkl5d3hXTkhOUHdLTUFvR0NDcUdTTTQ5QkFNQ0E0R0xBRENCaHdKQ0FOY1F4bWVRNG44elkybHFydGpobzlNUUttYll1T3pvV3o1Sm9cLzRkKzlPT1JaMFU5UTB6OEQrSUV0S1Q0ZGREZm9VTDBiMG9DR09WN08weGMzanpMbEFOQWtFOGs0dlYwODdjYjRaNktYMlF0TkVIZjFxWW95RXliNVFLWW51OGtqRmt2RmtoUTdWcTNHRFFGM2RHa0wyNkZFYVNMMGc2Q3ZwWUd6YjNlXC9jcVdvekY1Zz09Il0sIngiOiJBWndEQU5WU1hQNWVOd09WOThaOWFxek5fd0haQVVpOGFqdWMwcFNtMGxJSTV2QU1wU0V2a3liVHpTV0VkX2RSRFB1UmJuRzFxd3VSeER6QklxV29jSEc2IiwieSI6IkFHMGNsZGhMVkNsNHZWM1Q4OVBVQUw5ZEdSYjE4dVdud1RVT1lidTljOFp5dUU3OVlPd2ZqSUpzcUtBX1BCY2NwaTJEZzM1MTlvNlMySXl3eFdOSE5Qd0sifSx7Imt0eSI6IkVDIiwieDV0I1MyNTYiOiJpNHktOXplLWtqRXFnZjU3SjFFSnRtLXAyMXVXcnVtQ1lZY3pwUjhPQ3dFIiwiY3J2IjoiUC0yNTYiLCJraWQiOiJpNHkrOXplK2tqRT0iLCJ4NWMiOlsiTUlJQllUQ0NBUWlnQXdJQkFnSUVYRzUzVFRBS0JnZ3Foa2pPUFFRREFqQTVNUXN3Q1FZRFZRUUdFd0pUUlRFWU1CWUdBMVVFQ2d3UFNVUnpaV01nVTI5c2RYUnBiMjV6TVJBd0RnWURWUVFEREFkRlF5QjBaWE4wTUI0WERURTVNREl5TVRFd01ESTFNMW9YRFRJd01ESXlNVEV3TURJMU0xb3dPVEVMTUFrR0ExVUVCaE1DVTBVeEdEQVdCZ05WQkFvTUQwbEVjMlZqSUZOdmJIVjBhVzl1Y3pFUU1BNEdBMVVFQXd3SFJVTWdkR1Z6ZERCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQlAwXC9pa2JhdnlWcGZPOCtUTzF4Q080RTlSMU91THlnWnJEeCsyTG1MNUJlQUE1WnlOTmZ5b2IwT0UrTDBcL2tGczZ4eVNHR1gyUTFJUm9CMktJb1ZTcDR3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnUkEzd3dScG9uamZOWE5RY0Q0QzUydndrWXY0NmZqc08zZlJUNENQV2NnMENJRm1KMTRPVFBzMDlqTUNDaUhOTElLekhxNVFieGxCUmZEUWRDXC9BRW9pelIiXSwieCI6Il9ULUtSdHFfSldsODd6NU03WEVJN2dUMUhVNjR2S0Jtc1BIN1l1WXZrRjQiLCJ5IjoiQUE1WnlOTmZ5b2IwT0UtTDBfa0ZzNnh5U0dHWDJRMUlSb0IyS0lvVlNwNCJ9XX19LCJpc3MiOiJodHRwczpcL1wvZGdjLmRpZ2cuc2VcL3RydXN0LXNlcnZpY2UiLCJpZCI6ImY2NjdjYThjZDg5NTY3ZTYzMDU3OGMwMDRiN2U0OTE4IiwiZXhwIjoxNjIwMTE3MzM3LCJpYXQiOjE2MjAwMzA5Mzd9.8RNpblJ217KCqztC77LldugxlusgyPYxbFbz2cbgfDZ274cWUtTdUef70cyScHOVABINstR-nMuNHpKBtf8yLA
```



### 4.2. Header Example

Decoded header

```
{
  "x5c" : [ "MIIBYTCCAQigAwIBAgIEXG53TTAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMRAwDgYDVQQDDAdFQyB0ZXN0MB4XDTE5MDIyMTEwMDI1M1oXDTIwMDIyMTEwMDI1M1owOTELMAkGA1UEBhMCU0UxGDAWBgNVBAoMD0lEc2VjIFNvbHV0aW9uczEQMA4GA1UEAwwHRUMgdGVzdDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABP0/ikbavyVpfO8+TO1xCO4E9R1OuLygZrDx+2LmL5BeAA5ZyNNfyob0OE+L0/kFs6xySGGX2Q1IRoB2KIoVSp4wCgYIKoZIzj0EAwIDRwAwRAIgRA3wwRponjfNXNQcD4C52vwkYv46fjsO3fRT4CPWcg0CIFmJ14OTPs09jMCCiHNLIKzHq5QbxlBRfDQdC/AEoizR" ],
  "typ" : "JOSE",
  "alg" : "ES256"
}

```
### 4.3. Payload Example

Decoded payload

```
{
  "iss" : "https://dgc.digg.se/trust-service",
  "id" : "f667ca8cd89567e630578c004b7e4918",
  "exp" : 1620117337,
  "iat" : 1620030937,
  "aud" : [ "dgc-validators" ],
  "dsc_trust_list" : {
    "DE" : {
      "eku" : {
        "i4y+9ze+kjE=" : [ "1.3.6.1.4.1.0.1847.2021.1.1" ],
        "WTvX+br9/Pc=" : [ "1.3.6.1.4.1.0.1847.2021.1.2" ]
      },
      "keys" : [ {
        "kty" : "EC",
        "x5t#S256" : "i4y-9ze-kjEqgf57J1EJtm-p21uWrumCYYczpR8OCwE",
        "crv" : "P-256",
        "kid" : "i4y+9ze+kjE=",
        "x5c" : [ "MIIBYTCCAQigAwIBAgIEXG53TTAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMRAwDgYDVQQDDAdFQyB0ZXN0MB4XDTE5MDIyMTEwMDI1M1oXDTIwMDIyMTEwMDI1M1owOTELMAkGA1UEBhMCU0UxGDAWBgNVBAoMD0lEc2VjIFNvbHV0aW9uczEQMA4GA1UEAwwHRUMgdGVzdDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABP0/ikbavyVpfO8+TO1xCO4E9R1OuLygZrDx+2LmL5BeAA5ZyNNfyob0OE+L0/kFs6xySGGX2Q1IRoB2KIoVSp4wCgYIKoZIzj0EAwIDRwAwRAIgRA3wwRponjfNXNQcD4C52vwkYv46fjsO3fRT4CPWcg0CIFmJ14OTPs09jMCCiHNLIKzHq5QbxlBRfDQdC/AEoizR" ],
        "x" : "_T-KRtq_JWl87z5M7XEI7gT1HU64vKBmsPH7YuYvkF4",
        "y" : "AA5ZyNNfyob0OE-L0_kFs6xySGGX2Q1IRoB2KIoVSp4"
      }, {
        "kty" : "EC",
        "x5t#S256" : "iOqgTe_pun8LcBf9r0mdmzGLNcGjqfww4btZWVZt5vI",
        "crv" : "P-521",
        "kid" : "iOqgTe/pun8=",
        "x5c" : [ "MIIB6TCCAUugAwIBAgIEXHAXuDAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEOMAwGA1UECgwFSURzZWMxGjAYBgNVBAMMEU9wZW5TQU1MIEVDQyBUZXN0MB4XDTE5MDIyMjE1MzkzNloXDTIwMDIyMjE1MzkzNlowOTELMAkGA1UEBhMCU0UxDjAMBgNVBAoMBUlEc2VjMRowGAYDVQQDDBFPcGVuU0FNTCBFQ0MgVGVzdDCBmzAQBgcqhkjOPQIBBgUrgQQAIwOBhgAEAZwDANVSXP5eNwOV98Z9aqzN/wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd/dRDPuRbnG1qwuRxDzBIqWocHG6AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA/PBccpi2Dg3519o6S2IywxWNHNPwKMAoGCCqGSM49BAMCA4GLADCBhwJCANcQxmeQ4n8zY2lqrtjho9MQKmbYuOzoWz5Jo/4d+9OORZ0U9Q0z8D+IEtKT4ddDfoUL0b0oCGOV7O0xc3jzLlANAkE8k4vV087cb4Z6KX2QtNEHf1qYoyEyb5QKYnu8kjFkvFkhQ7Vq3GDQF3dGkL26FEaSL0g6CvpYGzb3e/cqWozF5g==" ],
        "x" : "AZwDANVSXP5eNwOV98Z9aqzN_wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd_dRDPuRbnG1qwuRxDzBIqWocHG6",
        "y" : "AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA_PBccpi2Dg3519o6S2IywxWNHNPwK"
      }, {
        "kty" : "RSA",
        "x5t#S256" : "WTvX-br9_PfwBjnuxuwVd3P6OuiAaLjr2HwNBjBFYEc",
        "e" : "AQAB",
        "use" : "sig",
        "kid" : "WTvX+br9/Pc=",
        "x5c" : [ "MIIEJDCCAoygAwIBAgIEXmJSrDANBgkqhkiG9w0BAQsFADBGMQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMR0wGwYDVQQDDBRKb2huIERvZSBUZXN0IFNpZ25lcjAeFw0yMDAzMDYxMzM5NTZaFw0yMTAzMDYxMzM5NTZaMEYxCzAJBgNVBAYTAlNFMRgwFgYDVQQKDA9JRHNlYyBTb2x1dGlvbnMxHTAbBgNVBAMMFEpvaG4gRG9lIFRlc3QgU2lnbmVyMIIBojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEAslDF2crOLWCY46CetcUtI1Pdzcc2VkBJpcrIweonj4h4Z0tFXOrcjoU+wfnli3P195YJftGYRqgQcIwu9FF8az589AH7RyC82IvobhnhHLXwDMTHOe7ykpZ2tC/PWq9Rg20rdE4rFocnX//1HyzZ5JjwV1/NyHbucx7tURHpnxx+qOlGn7R2efpzLYvokFAQSVKszxaMj4O+lx6Lt83ULHoRZjDxUlf9kJaKELeWAsqgHl4Vbry40tRXSafPwNvSirNcxt8dV9IuPI13Lz+zOKgJlDZikX5+RP+Ur74LJV9PF8F9Z7CPEZAuTomS1MEjGW5x4Zgpj7UvMZ79KgxHOl1TZk0B94HoMdPyLH6HQ4S9S8yy9Oxh4wqq7y36wlPTd5dQ6NrShKGhSrMmw31RMGqZRcl0im7g0rsUYl5eQcRT1uKgzDp3nESdx/+0VhVv3ERoDm6Juz28OLy+bHNtPT6EQKSmnDJt4syZE7BaXqM6QiByi1lhDLTSREWE5jntAgMBAAGjGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgXgMA0GCSqGSIb3DQEBCwUAA4IBgQCr4mI7T4l/O4ER20bgo7Qe/2OyYkv7BKFuGUFYeoFXLqsU3huLPxsQQVjZ+igK1o6OoHWT6pcoQ2G6dz6AJfghBeZZOl0NFMZ8wv/5AKb4yMAnNSQMS9+vk/i8AOQD3RXuKVS4i2zC0pAk/UZVk3g4wVs8c2ZYcRyxAClJNoiHifjjHtuGSOEBZJWt22L3IO+pjcTy3lUA31cxUo+J3F+IYaTTEhdNEiHqHmAXVq280AJnRJeTWvVrkUri9/Nygo+Fzx95yay5Duau5DfcdQ3ar7U1CL34PO24L9mBGANPvyPHW8kjvZVmJo0LybXIdR5FSG8ya+SOXnCvFUbiSrmrfoHugck8N4oBD8MX6pGQIUTuuRARnpMQsm5t1eF7fH+SNDRh3QqB89vF71ErjrQTm5FDO8Vz3Hn68YajxTekaafHhrC1G9FE2v1iBrENx68LeF4C0FJpCFlAVDYKKfikQmSRM4APmxQsKW+T68WC4s3kQmT4LZEI+A9/ESraXoE=" ],
        "n" : "slDF2crOLWCY46CetcUtI1Pdzcc2VkBJpcrIweonj4h4Z0tFXOrcjoU-wfnli3P195YJftGYRqgQcIwu9FF8az589AH7RyC82IvobhnhHLXwDMTHOe7ykpZ2tC_PWq9Rg20rdE4rFocnX__1HyzZ5JjwV1_NyHbucx7tURHpnxx-qOlGn7R2efpzLYvokFAQSVKszxaMj4O-lx6Lt83ULHoRZjDxUlf9kJaKELeWAsqgHl4Vbry40tRXSafPwNvSirNcxt8dV9IuPI13Lz-zOKgJlDZikX5-RP-Ur74LJV9PF8F9Z7CPEZAuTomS1MEjGW5x4Zgpj7UvMZ79KgxHOl1TZk0B94HoMdPyLH6HQ4S9S8yy9Oxh4wqq7y36wlPTd5dQ6NrShKGhSrMmw31RMGqZRcl0im7g0rsUYl5eQcRT1uKgzDp3nESdx_-0VhVv3ERoDm6Juz28OLy-bHNtPT6EQKSmnDJt4syZE7BaXqM6QiByi1lhDLTSREWE5jnt"
      } ]
    },
    "SE" : {
      "eku" : {
        "i4y+9ze+kjE=" : [ "1.3.6.1.4.1.0.1847.2021.1.2", "1.3.6.1.4.1.0.1847.2021.1.3" ],
        "iOqgTe/pun8=" : [ "1.3.6.1.4.1.0.1847.2021.1.1" ]
      },
      "keys" : [ {
        "kty" : "EC",
        "x5t#S256" : "iOqgTe_pun8LcBf9r0mdmzGLNcGjqfww4btZWVZt5vI",
        "crv" : "P-521",
        "kid" : "iOqgTe/pun8=",
        "x5c" : [ "MIIB6TCCAUugAwIBAgIEXHAXuDAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEOMAwGA1UECgwFSURzZWMxGjAYBgNVBAMMEU9wZW5TQU1MIEVDQyBUZXN0MB4XDTE5MDIyMjE1MzkzNloXDTIwMDIyMjE1MzkzNlowOTELMAkGA1UEBhMCU0UxDjAMBgNVBAoMBUlEc2VjMRowGAYDVQQDDBFPcGVuU0FNTCBFQ0MgVGVzdDCBmzAQBgcqhkjOPQIBBgUrgQQAIwOBhgAEAZwDANVSXP5eNwOV98Z9aqzN/wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd/dRDPuRbnG1qwuRxDzBIqWocHG6AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA/PBccpi2Dg3519o6S2IywxWNHNPwKMAoGCCqGSM49BAMCA4GLADCBhwJCANcQxmeQ4n8zY2lqrtjho9MQKmbYuOzoWz5Jo/4d+9OORZ0U9Q0z8D+IEtKT4ddDfoUL0b0oCGOV7O0xc3jzLlANAkE8k4vV087cb4Z6KX2QtNEHf1qYoyEyb5QKYnu8kjFkvFkhQ7Vq3GDQF3dGkL26FEaSL0g6CvpYGzb3e/cqWozF5g==" ],
        "x" : "AZwDANVSXP5eNwOV98Z9aqzN_wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd_dRDPuRbnG1qwuRxDzBIqWocHG6",
        "y" : "AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA_PBccpi2Dg3519o6S2IywxWNHNPwK"
      }, {
        "kty" : "EC",
        "x5t#S256" : "i4y-9ze-kjEqgf57J1EJtm-p21uWrumCYYczpR8OCwE",
        "crv" : "P-256",
        "kid" : "i4y+9ze+kjE=",
        "x5c" : [ "MIIBYTCCAQigAwIBAgIEXG53TTAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMRAwDgYDVQQDDAdFQyB0ZXN0MB4XDTE5MDIyMTEwMDI1M1oXDTIwMDIyMTEwMDI1M1owOTELMAkGA1UEBhMCU0UxGDAWBgNVBAoMD0lEc2VjIFNvbHV0aW9uczEQMA4GA1UEAwwHRUMgdGVzdDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABP0/ikbavyVpfO8+TO1xCO4E9R1OuLygZrDx+2LmL5BeAA5ZyNNfyob0OE+L0/kFs6xySGGX2Q1IRoB2KIoVSp4wCgYIKoZIzj0EAwIDRwAwRAIgRA3wwRponjfNXNQcD4C52vwkYv46fjsO3fRT4CPWcg0CIFmJ14OTPs09jMCCiHNLIKzHq5QbxlBRfDQdC/AEoizR" ],
        "x" : "_T-KRtq_JWl87z5M7XEI7gT1HU64vKBmsPH7YuYvkF4",
        "y" : "AA5ZyNNfyob0OE-L0_kFs6xySGGX2Q1IRoB2KIoVSp4"
      } ]
    }
  }
}
```
