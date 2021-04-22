# Document Signer Certificate (DSC) Trust List - Format Specification

## Abstract

This document specifies the format for Document Signer Certificate (DSC) Trust List (DSC-TL).

The DSC-TL contains trusted DSC certificates for one or more countries representing Digital Green Certificate (DGC) Document Signers that are trusted according to the EU DGC Gateway.

## Terminology

Organisations adopting this specification for issuing health certificates are called `Document Signers` and organisations accepting health certificates as proof of health status are called `Verifiers`. Together, these are called `Participants`. `Document Signers` are registered under a country and `Verifiers` are assumed to be operating within a country infrastructure, relying on a national `Trust Point` for retrieving the information necessary to validate the authenticity of `Gigital Green Certificates` (`DGC`) issued by `Document Signers` in other countries. The national `Trust Points` are operated by a competent appointed authority of that country. Countries operating together to form this infrastructure of trust are called `Member States`. Some aspects in this document must be coordinated between the `Member States`, such as the management of a namespace and the distribution of cryptographic keys. It is assumed that a function, hereafter referred to as the `DGC Gateway`, carries out these tasks.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 ([RFC2119](https://tools.ietf.org/html/rfc2119), [RFC8174](https://tools.ietf.org/html/rfc8174)) when, and only when, they appear in all capitals, as shown here.

## 1. Introduction

The EU Gateway provide an API that allows Trust Points in Member States to upload and down load certificates used to support validation of Digital Green Certificates (DGC). Each Member State operates at least one Courtry Signing Certificate Authority (CSCA) that issues certificates for national Document Signers. These Document Signer Certificates are shared by the Trust Point with all national Verifiers.

This specification specifies the Swedish national infrastructure data format for distributing a list of trusted DSC, making up the DSC Trust List (DSC-TL).

## 2. DSC-TL Format

The DSC-TL format is contained in a signed JSON Web Signature (JWS) according to [[RFC 7515](https://tools.ietf.org/html/rfc7515)].

The key data is carried as unencoded payload according to [[RFC 7797](https://tools.ietf.org/html/rfc7797)]. The main reason for this is that the unencoded payload is significantly smaller in size, where the base64 enocded payload offers no advantages to motivate its larger size for the use of JWS according to this specification.
As unencoded payload is only allowed in JWS, and not JSON Web Token (JWT) the design choice is therefore to use JWS instead of a JWT.

### 2.1. JWS Header

The JWS header make use of the following header parameters:

Header Parameter | Value/Description
---|---
`typ`  | Set to the value `JOSE` to indicate that this is a JWS
`b64`  | As specified in  [[RFC 7797](https://tools.ietf.org/html/rfc7797)] is set to the value `false` to indicate unencoded payload.
`alg` | Specifies the algorithm used to sign the JWS
`x5c` | Carries the signing certificate and optionally any supporting certificate that may be used to validate the signing certificate.

### 2.2. JWS Payload

The payload holds a map where each country data is keyd under its 2 letter ISO 3166 country code.
The object under each country holds 2 objects:

Object | Description
--- | ---
eku  | A map keyed under each DSC key ID (kid) expressing an array of Extended Key Usage OID (Object identifiers) defining the usage contraints for that DSC.
keys  |  The JWK [[RFC7517](https://tools.ietf.org/html/rfc7517)] key data for each DSC valid for the specified country.

#### 2.2.1 Design Considerations

The reason why the eku data is placed outside of the JWK set is to align with common implementations of JWK and to ensure the highest possible degree of compatibility with major software libraries. Even if the standard for JWK allows cusomized object items in the JWK, implementations such as Nimbus lacks the capability to natively add or parse such additional parameters.

Each DCS includes the EKU data, which can easily be extracted by standard tools. The "eku" is present as an optional resource for those implementations that would prefer not to parse any X.509 certificate ASN.1.

Keying material is provided as public key material as well as in the form of a complete DSC X.509 certificate. The certificate is considered the normative representation of the key as it also may contain metadata for the key such as the issuer name and usage constraning identifiers.

## 3. Schema

The following JSON schema defines the structure of the DSC-TL payload:

```
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "DSC-TL format JSON Schema",
    "description": "Schema defining the payload format for Document Signing Certificate - Trust List\n        information",
    "type": "object",
    "required": [
        "iss",
        "iat",
        "exp",
        "dsc_trust_list"
    ],
    "properties": {
        "iss": {
            "description": "Identifier of the issuer of the DSC-TL",
            "type": "string"
        },
        "iat": {
            "description": "Issued at time",
            "type": "integer"
        },
        "exp": {
            "description": "Expirey time",
            "type": "integer"
        },
        "dsc_trust_list": {
            "description": "List of trusted DSC under each country ISO3166 2 letter code as key",
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

### 4.1 DSC-TL example

```
eyJiNjQiOmZhbHNlLCJ4NWMiOlsiTUlJQllUQ0NBUWlnQXdJQkFnSUVYRzUzVFRBS0JnZ3Foa2pPUFFRREFqQTVNUXN3Q1FZRFZRUUdFd0pUUlRFWU1CWUdBMVVFQ2d3UFNVUnpaV01nVTI5c2RYUnBiMjV6TVJBd0RnWURWUVFEREFkRlF5QjBaWE4wTUI0WERURTVNREl5TVRFd01ESTFNMW9YRFRJd01ESXlNVEV3TURJMU0xb3dPVEVMTUFrR0ExVUVCaE1DVTBVeEdEQVdCZ05WQkFvTUQwbEVjMlZqSUZOdmJIVjBhVzl1Y3pFUU1BNEdBMVVFQXd3SFJVTWdkR1Z6ZERCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQlAwXC9pa2JhdnlWcGZPOCtUTzF4Q080RTlSMU91THlnWnJEeCsyTG1MNUJlQUE1WnlOTmZ5b2IwT0UrTDBcL2tGczZ4eVNHR1gyUTFJUm9CMktJb1ZTcDR3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnUkEzd3dScG9uamZOWE5RY0Q0QzUydndrWXY0NmZqc08zZlJUNENQV2NnMENJRm1KMTRPVFBzMDlqTUNDaUhOTElLekhxNVFieGxCUmZEUWRDXC9BRW9pelIiXSwidHlwIjoiSk9TRSIsImFsZyI6IkVTMjU2In0.{"aud":"dgc-validators","dsc_trust_list":{"DE":{"eku":{"i4y+9ze+kjE=":["1.3.6.1.4.1.0.1847.2021.1.1"],"WTvX+br9\/Pc=":["1.3.6.1.4.1.0.1847.2021.1.2"]},"keys":[{"kty":"EC","x5t#S256":"i4y-9ze-kjEqgf57J1EJtm-p21uWrumCYYczpR8OCwE","crv":"P-256","kid":"i4y+9ze+kjE=","x5c":["MIIBYTCCAQigAwIBAgIEXG53TTAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMRAwDgYDVQQDDAdFQyB0ZXN0MB4XDTE5MDIyMTEwMDI1M1oXDTIwMDIyMTEwMDI1M1owOTELMAkGA1UEBhMCU0UxGDAWBgNVBAoMD0lEc2VjIFNvbHV0aW9uczEQMA4GA1UEAwwHRUMgdGVzdDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABP0\/ikbavyVpfO8+TO1xCO4E9R1OuLygZrDx+2LmL5BeAA5ZyNNfyob0OE+L0\/kFs6xySGGX2Q1IRoB2KIoVSp4wCgYIKoZIzj0EAwIDRwAwRAIgRA3wwRponjfNXNQcD4C52vwkYv46fjsO3fRT4CPWcg0CIFmJ14OTPs09jMCCiHNLIKzHq5QbxlBRfDQdC\/AEoizR"],"x":"_T-KRtq_JWl87z5M7XEI7gT1HU64vKBmsPH7YuYvkF4","y":"AA5ZyNNfyob0OE-L0_kFs6xySGGX2Q1IRoB2KIoVSp4"},{"kty":"EC","x5t#S256":"iOqgTe_pun8LcBf9r0mdmzGLNcGjqfww4btZWVZt5vI","crv":"P-521","kid":"iOqgTe\/pun8=","x5c":["MIIB6TCCAUugAwIBAgIEXHAXuDAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEOMAwGA1UECgwFSURzZWMxGjAYBgNVBAMMEU9wZW5TQU1MIEVDQyBUZXN0MB4XDTE5MDIyMjE1MzkzNloXDTIwMDIyMjE1MzkzNlowOTELMAkGA1UEBhMCU0UxDjAMBgNVBAoMBUlEc2VjMRowGAYDVQQDDBFPcGVuU0FNTCBFQ0MgVGVzdDCBmzAQBgcqhkjOPQIBBgUrgQQAIwOBhgAEAZwDANVSXP5eNwOV98Z9aqzN\/wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd\/dRDPuRbnG1qwuRxDzBIqWocHG6AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA\/PBccpi2Dg3519o6S2IywxWNHNPwKMAoGCCqGSM49BAMCA4GLADCBhwJCANcQxmeQ4n8zY2lqrtjho9MQKmbYuOzoWz5Jo\/4d+9OORZ0U9Q0z8D+IEtKT4ddDfoUL0b0oCGOV7O0xc3jzLlANAkE8k4vV087cb4Z6KX2QtNEHf1qYoyEyb5QKYnu8kjFkvFkhQ7Vq3GDQF3dGkL26FEaSL0g6CvpYGzb3e\/cqWozF5g=="],"x":"AZwDANVSXP5eNwOV98Z9aqzN_wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd_dRDPuRbnG1qwuRxDzBIqWocHG6","y":"AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA_PBccpi2Dg3519o6S2IywxWNHNPwK"},{"kty":"RSA","x5t#S256":"WTvX-br9_PfwBjnuxuwVd3P6OuiAaLjr2HwNBjBFYEc","e":"AQAB","use":"sig","kid":"WTvX+br9\/Pc=","x5c":["MIIEJDCCAoygAwIBAgIEXmJSrDANBgkqhkiG9w0BAQsFADBGMQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMR0wGwYDVQQDDBRKb2huIERvZSBUZXN0IFNpZ25lcjAeFw0yMDAzMDYxMzM5NTZaFw0yMTAzMDYxMzM5NTZaMEYxCzAJBgNVBAYTAlNFMRgwFgYDVQQKDA9JRHNlYyBTb2x1dGlvbnMxHTAbBgNVBAMMFEpvaG4gRG9lIFRlc3QgU2lnbmVyMIIBojANBgkqhkiG9w0BAQEFAAOCAY8AMIIBigKCAYEAslDF2crOLWCY46CetcUtI1Pdzcc2VkBJpcrIweonj4h4Z0tFXOrcjoU+wfnli3P195YJftGYRqgQcIwu9FF8az589AH7RyC82IvobhnhHLXwDMTHOe7ykpZ2tC\/PWq9Rg20rdE4rFocnX\/\/1HyzZ5JjwV1\/NyHbucx7tURHpnxx+qOlGn7R2efpzLYvokFAQSVKszxaMj4O+lx6Lt83ULHoRZjDxUlf9kJaKELeWAsqgHl4Vbry40tRXSafPwNvSirNcxt8dV9IuPI13Lz+zOKgJlDZikX5+RP+Ur74LJV9PF8F9Z7CPEZAuTomS1MEjGW5x4Zgpj7UvMZ79KgxHOl1TZk0B94HoMdPyLH6HQ4S9S8yy9Oxh4wqq7y36wlPTd5dQ6NrShKGhSrMmw31RMGqZRcl0im7g0rsUYl5eQcRT1uKgzDp3nESdx\/+0VhVv3ERoDm6Juz28OLy+bHNtPT6EQKSmnDJt4syZE7BaXqM6QiByi1lhDLTSREWE5jntAgMBAAGjGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgXgMA0GCSqGSIb3DQEBCwUAA4IBgQCr4mI7T4l\/O4ER20bgo7Qe\/2OyYkv7BKFuGUFYeoFXLqsU3huLPxsQQVjZ+igK1o6OoHWT6pcoQ2G6dz6AJfghBeZZOl0NFMZ8wv\/5AKb4yMAnNSQMS9+vk\/i8AOQD3RXuKVS4i2zC0pAk\/UZVk3g4wVs8c2ZYcRyxAClJNoiHifjjHtuGSOEBZJWt22L3IO+pjcTy3lUA31cxUo+J3F+IYaTTEhdNEiHqHmAXVq280AJnRJeTWvVrkUri9\/Nygo+Fzx95yay5Duau5DfcdQ3ar7U1CL34PO24L9mBGANPvyPHW8kjvZVmJo0LybXIdR5FSG8ya+SOXnCvFUbiSrmrfoHugck8N4oBD8MX6pGQIUTuuRARnpMQsm5t1eF7fH+SNDRh3QqB89vF71ErjrQTm5FDO8Vz3Hn68YajxTekaafHhrC1G9FE2v1iBrENx68LeF4C0FJpCFlAVDYKKfikQmSRM4APmxQsKW+T68WC4s3kQmT4LZEI+A9\/ESraXoE="],"n":"slDF2crOLWCY46CetcUtI1Pdzcc2VkBJpcrIweonj4h4Z0tFXOrcjoU-wfnli3P195YJftGYRqgQcIwu9FF8az589AH7RyC82IvobhnhHLXwDMTHOe7ykpZ2tC_PWq9Rg20rdE4rFocnX__1HyzZ5JjwV1_NyHbucx7tURHpnxx-qOlGn7R2efpzLYvokFAQSVKszxaMj4O-lx6Lt83ULHoRZjDxUlf9kJaKELeWAsqgHl4Vbry40tRXSafPwNvSirNcxt8dV9IuPI13Lz-zOKgJlDZikX5-RP-Ur74LJV9PF8F9Z7CPEZAuTomS1MEjGW5x4Zgpj7UvMZ79KgxHOl1TZk0B94HoMdPyLH6HQ4S9S8yy9Oxh4wqq7y36wlPTd5dQ6NrShKGhSrMmw31RMGqZRcl0im7g0rsUYl5eQcRT1uKgzDp3nESdx_-0VhVv3ERoDm6Juz28OLy-bHNtPT6EQKSmnDJt4syZE7BaXqM6QiByi1lhDLTSREWE5jnt"}]},"SE":{"eku":{"i4y+9ze+kjE=":["1.3.6.1.4.1.0.1847.2021.1.2","1.3.6.1.4.1.0.1847.2021.1.3"],"iOqgTe\/pun8=":["1.3.6.1.4.1.0.1847.2021.1.1"]},"keys":[{"kty":"EC","x5t#S256":"iOqgTe_pun8LcBf9r0mdmzGLNcGjqfww4btZWVZt5vI","crv":"P-521","kid":"iOqgTe\/pun8=","x5c":["MIIB6TCCAUugAwIBAgIEXHAXuDAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEOMAwGA1UECgwFSURzZWMxGjAYBgNVBAMMEU9wZW5TQU1MIEVDQyBUZXN0MB4XDTE5MDIyMjE1MzkzNloXDTIwMDIyMjE1MzkzNlowOTELMAkGA1UEBhMCU0UxDjAMBgNVBAoMBUlEc2VjMRowGAYDVQQDDBFPcGVuU0FNTCBFQ0MgVGVzdDCBmzAQBgcqhkjOPQIBBgUrgQQAIwOBhgAEAZwDANVSXP5eNwOV98Z9aqzN\/wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd\/dRDPuRbnG1qwuRxDzBIqWocHG6AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA\/PBccpi2Dg3519o6S2IywxWNHNPwKMAoGCCqGSM49BAMCA4GLADCBhwJCANcQxmeQ4n8zY2lqrtjho9MQKmbYuOzoWz5Jo\/4d+9OORZ0U9Q0z8D+IEtKT4ddDfoUL0b0oCGOV7O0xc3jzLlANAkE8k4vV087cb4Z6KX2QtNEHf1qYoyEyb5QKYnu8kjFkvFkhQ7Vq3GDQF3dGkL26FEaSL0g6CvpYGzb3e\/cqWozF5g=="],"x":"AZwDANVSXP5eNwOV98Z9aqzN_wHZAUi8ajuc0pSm0lII5vAMpSEvkybTzSWEd_dRDPuRbnG1qwuRxDzBIqWocHG6","y":"AG0cldhLVCl4vV3T89PUAL9dGRb18uWnwTUOYbu9c8ZyuE79YOwfjIJsqKA_PBccpi2Dg3519o6S2IywxWNHNPwK"},{"kty":"EC","x5t#S256":"i4y-9ze-kjEqgf57J1EJtm-p21uWrumCYYczpR8OCwE","crv":"P-256","kid":"i4y+9ze+kjE=","x5c":["MIIBYTCCAQigAwIBAgIEXG53TTAKBggqhkjOPQQDAjA5MQswCQYDVQQGEwJTRTEYMBYGA1UECgwPSURzZWMgU29sdXRpb25zMRAwDgYDVQQDDAdFQyB0ZXN0MB4XDTE5MDIyMTEwMDI1M1oXDTIwMDIyMTEwMDI1M1owOTELMAkGA1UEBhMCU0UxGDAWBgNVBAoMD0lEc2VjIFNvbHV0aW9uczEQMA4GA1UEAwwHRUMgdGVzdDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABP0\/ikbavyVpfO8+TO1xCO4E9R1OuLygZrDx+2LmL5BeAA5ZyNNfyob0OE+L0\/kFs6xySGGX2Q1IRoB2KIoVSp4wCgYIKoZIzj0EAwIDRwAwRAIgRA3wwRponjfNXNQcD4C52vwkYv46fjsO3fRT4CPWcg0CIFmJ14OTPs09jMCCiHNLIKzHq5QbxlBRfDQdC\/AEoizR"],"x":"_T-KRtq_JWl87z5M7XEI7gT1HU64vKBmsPH7YuYvkF4","y":"AA5ZyNNfyob0OE-L0_kFs6xySGGX2Q1IRoB2KIoVSp4"}]}},"iss":"https:\/\/dgc.digg.se\/trust-service","id":"ddca935addee34f0cde2c0131b3bf7ea","exp":1619210638,"iat":1619124238}.P7xw7xcZuO1XmWVhVUYLeSXHZF5PBY040O0k2FheIpSOMnMBPw6NLBViWpjlWssEVpM5o_w0PlATYG3npNkLSA
```



### 4.2. Header example

Decoded header

```
{
  "b64" : false,
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
  "id" : "ddca935addee34f0cde2c0131b3bf7ea",
  "exp" : 1619210638,
  "iat" : 1619124238,
  "aud" : "dgc-validators",
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
