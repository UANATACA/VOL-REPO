# What it is

<div style="text-align: justify">
The high-performance solution for electronic signature Validation and Verification service, providing detailed information of the result.
<br></br>
Uanataca VOL API is a turnkey system that can be easily integrated with any business application or web thanks to its high-level web API based on the http RESTful paradigm.
</div>

# How it works

The solution is given with VOL Optimizer that is a server system exposing http RESTful APIs by means of which, business applications are enabled to require the electronic signature validation of documents.

VOL Optimizer performs the most computationally expensive workload of the validation and verification process, thus reducing the data traffic on the local network. The signed documents to be validated are processed in the customer business layer and are not send to Uanataca Services.

The system performs the validation and verification on all the signatures contained in a document, including timestamps. The validation consists on:

- Verify that the certificate is issued by a Trusted Service Provider (TSP).
- Verify that the certificate was valid at the time of signing. It means that the certificate was not expired or revoked at that moment.
- Verify the integrity of the document, verifying that no changes were done to the document after the signature.

The verification of the digital certificate is carried out through OCSP and CRL queries to the corresponding TSP issuer.

![img](https://github.com/UANATACA/VOL-REPO/raw/main/img/vol_hiw.png)


# Configuration

VOL Optimizer is supplied as a **Docker** image.


## Hardware requirements


**CPU:** modern multicore (minimum 4 core)

**RAM:** 8GB

**HDD:** 200 GB


## VOL Optimizer on Docker


This configuration requires a server with a Linux CentOS operating system.

</br>


> STEP 1: Install Docker and Docker-Compose.

*Docker*

Run the following commands in this order.

	sudo yum update -y
	yum install -y yum-utils device-mapper-persistent-data lvm2
	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	sudo yum install -y docker-ce docker-ce-cli containerd.io
	sudo systemctl start docker



*Docker-Compose*

Run the following commands in this order.


	sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
	chmod +x /usr/local/bin/docker-compose


Run command **docker-compose version** to check the installation. The outcome should show this information:


![img](https://github.com/UANATACA/VOL-REPO/raw/main/img/vol-docker1.png) 

</br>

> STEP 2: Extract and copy VOL Optimizer zip content to the server.

Extract all `vol_optimizer_docker.zip` content in a local folder.

Move VOL Optimizer folder to the path **/opt** in the server.

The outcome should look like this:

![img](https://github.com/UANATACA/VOL-REPO/raw/main/img/vol-docker2.png)

</br>


> STEP 3: Load One-Shot Docker images.

Run the following commands:

	cd /opt/vol_optimizer
	docker image load -i vol-api.tar.gz
	docker image load -i vol-gui.tar.gz
	docker image load -i vol-web.tar.gz

Remove image files:

	rm -rf /opt/vol_optimizer/vol-api.tar.gz
	rm -rf /opt/vol_optimizer/vol-gui.tar.gz
	rm -rf /opt/vol_optimizer/vol-web.tar.gz

</br>

> STEP 4: Launch the service.

Run this command:

	docker-compose up -d

</br>

> STEP 5: Service settings.

See <a href="#section/Configuration/Service-settings">service settings</a> configuration section.

</br>


## Service settings

<div style="text-align: justify">
Uanataca VOL exposes a custom website to validate documents. The website requires an SSL certificate and Billing credentials. Both requirements are also needed to use the service through API.
</div>


**Requirements:**

- Billing credentials for Uanataca production environment.
- SSL Certificate (.cer) and key (.key) files.


> STEP 1: SSL Certificate.

The SSL certificates are required to expose the validation web. If the web is not exposed, the SSL certificate demo files provided in the `vol_optimizer_docker.zip` can be used.

Move to the following server path:

	cd /home/bit4id/vol-volumes/vol-web-storage/

Copy SSL certificate files to this path. If you don't have you own certificate, the provided demo certificate can be used.

	cd /opt/vol_optimizer/demo_SSL_certificate/
	cp server.cer /home/bit4id/vol-volumes/vol-web-storage/
	cp server.key /home/bit4id/vol-volumes/vol-web-storage/ 


</br>

> STEP 2: Billing credentials.

Copy `vol-gui-passwd.tar` file to the path **/home/bit4id/** in the server.
Extract the content:

	cd /home/bit4id/
	tar -xvf vol-gui-passwd.tar.gz

Run the script to set Billing credentials (username and password):

	./passwd

</br>

> STEP 3: Launch the service.

Run the following commands:

	cd /opt/vol_optimizer/
	systemctl start docker
	docker-compose start

Check service status:

	docker-compose ps

	Name               Command               State                Ports
	--------------------------------------------------------------------------------
	api     /opt/bit4id/api/bin/uwsgi  ...   Up
	docs    /docker-entrypoint.sh sh / ...   Up      80/tcp, 8080/tcp
	gui     /opt/bit4id/gui/bin/uwsgi  ...   Up
	redis   docker-entrypoint.sh redis ...   Up      6379/tcp
	web     /opt/bit4id/nginx/entry_po ...   Up      0.0.0.0:443->443/tcp,:::443->44
    	                                             3/tcp, 0.0.0.0:80->80/tcp,:::80
                                                 	->80/tcp

All services must be UP.

</br>



# Endpoint URLs

The URL where is exposed the API is composed as follows:

	{host}/api/{resource}/

> host

The {host} is the IP or domain from the VOL Optimizer.
<br>

> resource

It is the name of the resource of our interest.

Each resource can also have path parameters and sub-resources that are defined in the <a href="#APIReference">API Reference</a> below.
<br>

This is an example of endpoint:

	https://vol.uanataca.com/api/documents/


# Authentication

VOL service requires authentication with the provided **Billing credentials**.
 
Each endpoint requires an authentication header before performing the requested action.
The header must containt the type (Basic) and a Base64 string that contains the Billing credentials in the format **username:password** in Base64.

	Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

This is an example with cURL:

	1 | curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" -X GET https://vol.uanataca.com/api/documents/06618cc5d920cb09/content


# JSON validation response

The validation response is a JSON object containig the following schema: 

	{
		"report": {...},
		"errors": [...],
		"data": {...},
		"messages": {...},
		"validations": {...},
		"profile": [...]
	}

where the objects attributes are:

> report

Contains general information about the signatures.

* `now` [string]: date & time from the verification
* `type` [string]: electronic signature profile under the European eIDAS regulation
	*  Enum: [PAdES, XAdES, CAdES]
* `document_uri` [string]: uploaded document uri
* `annotations` [array]: array containing data objects from each signature and timestamp

Signature object:

* `type` [string]: signature
	*  Enum: [signature, timestamp]
* `name` [string]: signer's name
* `reason` [string]: reason of the signature
* `location` [string]: location where the signature is performed
* `signature_time` [string]: date & time of the signature
* `signature` [object]: See <a href="#section/JSON-validation-response/#Signature-structure">signature structure</a> 
 
Timestamp object:

* `type` [string]: timestamp
	*  Enum: [signature, timestamp]
* `tst_info` [object]: timestamp information
* `content_digest` [string]: timestamp content digest
* `signature` [object]: See <a href="#section/JSON-validation-response/#Signature-structure">signature structure</a> 


> errors

List of errors found in the validation process. Each error is composed by:

* `where` [string]: reference where the error was found
* `what` [string]: identifier of the error message. See [list of messages](#messages)


> data

Report and document information.

* `lang` [string]: date & time from the verification
* `upload_time` [string]: date & time from the document upload
* `outformat` [string]: response format
	*  Enum: [json]
* `filename` [string]: uploadded file name
* `headers` [object]: header used in the HTTP POST request
* `document_hash_type` [string]: hash algorithm used
* `template` [string]: validation response template
	*  Enum: [report]
* `report_uri` [string]: path to retrieve the validation report
* `verification_date` [string]: date & time from the validation (optional)
* `documentid` [string]: document identifier
* `document_hash` [string]: complete document hash regarding the algorithm used


> messages

List of message descriptions. The complete list is included in every validation response.

**TIMESTAMP_VERIFIED**: "The document has not been modified after the application of the timestamp",<br>
**INVALID_CONTENT_TYPE_ATTRIBUTE**: "Digital signature attribute is not compliant (contentType)",<br>
**MULTIPLE_SIGNING_TIME_ATTRIBUTE**: "Verification date is declared multiple times (signingTime)",<br>
**REVOKED**: "Certificate revoked",<br>
**SIGNATURE_TIMESTAMP_OK**: "Valid timestamp",<br> 
**UNKNOWN_REVOCATION_INFO**: "Revocation information is not available",<br> 
**UNSUPPORTED_ALGORITHM**: "Unsupported signing algorithm",<br>
**UNTRUSTED**: "Untrusted CA certificate",<br>
**CE2011130EU**: "Signature complies with EC Decision 2011/130/EU",<br> 
**NOT_YET_VALID**: "Certificate is not yet valid",<br>
**NOTCE2011130EU**: "Signature not complies with EC Decision 2011/130/EU",<br> 
**UNKNOWN_ISSUER**: "Certificate issued by an unknown authority",<br>
**CONTENT_TIMESTAMP_OK**: "Valid timestamp",<br>
**INVALID_CERTIFICATE_REF**: "References to certificates are incorrect",<br> 
**SIGNATURE_TIMESTAMP_ERROR**: "Invalid timestamp",<br>
**SIGNING_KEY_LESS_THAN_2048_BITS**: "The signature key has a length of less than 2048 bits",<br> 
**NO_CA_BASIC_CONSTRAINTS**: "Certificate root without attributes of CA",<br>
**INDETERMINATE_VALIDATION_TIME**: "Verification date is undetermined",<br>
**SIGNED_DATA_NOT_FOUND**: "Signed data is not present",<br>
**NO_OCSP_SIGNING_USAGE**: "Certificate used to sign OCSP response is not suitable for the purpose",<br> 
**SIGNATURE_VERIFIED**: "The document has not been modified after signature",<br>
**INVALID_TIMESTAMP**: "Invalid timestamp",<br>
**CONTENT_TIMESTAMP_ERROR**: "Invalid timestamp",<br> 
**NO_CERTIFICATE_SIGNING_USAGE**: "Certificate issued by a CA not suitable for the purpose",<br> 
**XML_SCHEMA_MISMATCH**: "The XML file does not respect the XAdES/XMLDSIG schema",<br>
**SIGNING_CERTIFICATE_V1**: "The reference to the signature certificate uses an obsolete format (signingCertificate 1)",<br>
**NO_TIMESTAMPING_USAGE**: "The TSA certificate used to issue the timestamp is not suitable for the purpose",<br> 
**NO_CRL_SIGNING_USAGE**: "Certificate used to sign the CRL is not suitable for the purpose",<br>
**INVALID_SIGNATURE**: "Invalid digital signature",<br>
**TRUSTED**: "Trusted CA certificate",<br>
**SUSPENDED**: "Certificate suspended",<br>
**INVALID_SIGNING_CERTIFICATE_REF**: "Reference to the signature certificate is incorrect",<br> 
**WEAK_HASHING_ALGORITHM**"The document digest was calculated using an weak algorithm",<br>
**NO_SIGNING_CERTIFICATE**: "Signing certificate is not present",<br>
**EXPIRED**: "Certificate expired",<br>
**NO_SIGNING_CERTIFICATE_REF**: "Signing certificate reference is not present",<br> 
**SIGNING_CERTIFICATE_REF_FOUND**: "Reference to the signature certificate is valid"


> validation

Contains validations results for each signature and timestamp.

* `signatures` [array]: contains the results of the signtaure validations. See <a href="#section/JSON-validation-response/#Signature-validation-structure">signature validation structure</a> 
* `timestamps` [array]: contains the results of the timestamp validations. See <a href="#section/JSON-validation-response/#Timestamp-validation-structure">timestamp validation structure</a>
* `result` [string]: validation result
	* Enum: [SUCCESS, ERROR]


> profile

Profile used in the validation.

* `type` [string]: electronic signature profile, under the European eIDAS regulation, used in the validation
	*  Enum: [PAdES, XAdES, CAdES]
* `name` [string]: profile name
* `description` [string]: profile description


Example of a validation response:

```json
{
  "report": {
    "now": "2021-05-06T09:38:53.075351", 
    "type": "PAdES", 
    "document_uri": "/contents?documentid=217dfbf293c440cd", 
    "annotations": [
      {
        "type": "signature", 
        "name": "John Smith Lopez", 
        "reason": null, 
        "location": null, 
        "signature_time": "2021-05-06T11:35:01+02:00", 
        "signature": {
          "version": 1, 
          "digest_algorithm_identifiers": [
            {
              "oid": "2.16.840.1.101.3.4.2.1", 
              "name": "sha256", 
              "parameters": null
            }
          ], 
          "signatures": [...], 
          "certificates": [...], 
          "crls": [...], 
          "ocsps": [...]
        }
      }
    ]
  }, 
  "errors": [], 
  "data": {
    "lang": "en", 
    "upload_time": "2021-05-06T09:37:54.556609", 
    "outformat": "json", 
    "filename": "sample_JS.pdf", 
    "headers": {
      "CONTENT-LENGTH": "1097136", 
      "ACCEPT-ENCODING": "gzip, deflate, br", 
      "X-FORWARDED-FOR": "2.136.119.0", 
      "HOST": "vol.uanataca.com", 
      "ACCEPT": "*/*", 
      "USER-AGENT": "PostmanRuntime/7.28.0", 
      "CONNECTION": "close", 
      "COOKIE": "sessionid=7uba3t8qimqazlesudgo868lhvgbmo1s", 
      "CACHE-CONTROL": "no-cache", 
      "X-REAL-IP": "2.136.119.0", 
      "CONTENT-TYPE": "multipart/form-data; boundary=--------------------------915876127613387628723467", 
      "AUTHORIZATION": "Basic dGVzdEB1YW5hdGFjYS5jb206YmVsb3JhZG83NA==", 
      "POSTMAN-TOKEN": "a1462d78-6c4c-4042-a10f-6d421504eb3b"
    }, 
    "document_hash_type": "sha256", 
    "template": "report", 
    "report_uri": "/reports?documentid=217dfbf293c440cd", 
    "verification_date": null, 
    "documentid": "217dfbf293c440cd", 
    "document_hash": "217dfbf293c440cd45681504b8eed1dc340e933c7389569101c45b696a22ceae"
  }, 
  "messages": {
    "TIMESTAMP_VERIFIED": "The document has not been modified after the application of the timestamp", 
    "INVALID_CONTENT_TYPE_ATTRIBUTE": "Digital signature attribute is not compliant (contentType)", 
    "MULTIPLE_SIGNING_TIME_ATTRIBUTE": "Verification date is declared multiple times (signingTime)", 
    "REVOKED": "Certificate revoked", 
    "SIGNATURE_TIMESTAMP_OK": "Valid timestamp", 
    "UNKNOWN_REVOCATION_INFO": "Revocation information is not available", 
    "UNSUPPORTED_ALGORITHM": "Unsupported signing algorithm", 
    "UNTRUSTED": "Untrusted CA certificate", 
    "CE2011130EU": "Signature complies with EC Decision 2011/130/EU", 
    "NOT_YET_VALID": "Certificate is not yet valid", 
    "NOTCE2011130EU": "Signature not complies with EC Decision 2011/130/EU", 
    "UNKNOWN_ISSUER": "Certificate issued by an unknown authority", 
    "CONTENT_TIMESTAMP_OK": "Valid timestamp", 
    "INVALID_CERTIFICATE_REF": "References to certificates are incorrect", 
    "SIGNATURE_TIMESTAMP_ERROR": "Invalid timestamp", 
    "SIGNING_KEY_LESS_THAN_2048_BITS": "The signature key has a length of less than 2048 bits", 
    "NO_CA_BASIC_CONSTRAINTS": "Certificate root without attributes of CA", 
    "INDETERMINATE_VALIDATION_TIME": "Verification date is undetermined", 
    "SIGNED_DATA_NOT_FOUND": "Signed data is not present", 
    "NO_OCSP_SIGNING_USAGE": "Certificate used to sign OCSP response is not suitable for the purpose", 
    "SIGNATURE_VERIFIED": "The document has not been modified after signature", 
    "INVALID_TIMESTAMP": "Invalid timestamp", 
    "CONTENT_TIMESTAMP_ERROR": "Invalid timestamp", 
    "NO_CERTIFICATE_SIGNING_USAGE": "Certificate issued by a CA not suitable for the purpose", 
    "XML_SCHEMA_MISMATCH": "The XML file does not respect the XAdES/XMLDSIG schema", 
    "SIGNING_CERTIFICATE_V1": "The reference to the signature certificate uses an obsolete format (signingCertificate v1)", 
    "NO_TIMESTAMPING_USAGE": "The TSA certificate used to issue the timestamp is not suitable for the purpose", 
    "NO_CRL_SIGNING_USAGE": "Certificate used to sign the CRL is not suitable for the purpose", 
    "INVALID_SIGNATURE": "Invalid digital signature", 
    "TRUSTED": "Trusted CA certificate", 
    "SUSPENDED": "Certificate suspended", 
    "INVALID_SIGNING_CERTIFICATE_REF": "Reference to the signature certificate is incorrect", 
    "WEAK_HASHING_ALGORITHM": "The document digest was calculated using an weak algorithm", 
    "NO_SIGNING_CERTIFICATE": "Signing certificate is not present", 
    "EXPIRED": "Certificate expired", 
    "NO_SIGNING_CERTIFICATE_REF": "Signing certificate reference is not present", 
    "SIGNING_CERTIFICATE_REF_FOUND": "Reference to the signature certificate is valid"
  }, 
  "validation": {
    "signatures": [
      {
        "data": ".report.annotations[0].signature.signatures[0]", 
        "type": "signature", 
        "status": {
          "": [
            "SUCCESS:SIGNATURE_VERIFIED"
          ], 
          ".validation.signatures[0].certificates[1]": [
            "SUCCESS:TRUSTED"
          ], 
          ".validation.signatures[0].timestamps[0]": [
            "SUCCESS:SIGNATURE_TIMESTAMP_OK"
          ]
        }, 
        "time": "2021-05-06T09:35:00+00:00", 
        "timestamps": [
          {
            "data": ".report.annotations[0].signature.signatures[0].unsigned_attributes[0].value.signature.signatures[0]", 
            "type": "signature_timestamp", 
            "status": {
              "": [
                "SUCCESS:TIMESTAMP_VERIFIED"
              ], 
              ".validation.signatures[0].timestamps[0].certificates[1]": [
                "SUCCESS:TRUSTED"
              ]
            }, 
            "time": "2021-05-06T09:35:00+00:00", 
            "tst_info": ".report.annotations[0].signature.signatures[0].unsigned_attributes[0].value.tst_info", 
            "certificate_refs": {
              "v1": [], 
              "v2": [
                ".report.annotations[0].signature.certificates[2]"
              ]
            }, 
            "certificates": [...], 
            "content_depth": 0, 
            "signing_certificate": ".report.annotations[0].signature.certificates[2]", 
            "stamps": 0, 
            "index": 0
          }
        ], 
        "certificate_refs": {
          "v1": [], 
          "v2": []
        }, 
        "certificates": [...], 
        "content_depth": 0, 
        "signing_certificate": ".report.annotations[0].signature.certificates[0]", 
        "index": 0, 
        "name": "John Smith Lopez", 
        "reason": null, 
        "location": null, 
        "level": "PAdES-T"
      }
    ], 
    "timestamps": [], 
    "result": "SUCCESS"
  }, 
  "profile": [
    {
      "type": "PAdES", 
      "name": "PAdES", 
      "description": ""
    }
  ]
}
```

## Signature validation structure

* `data` [string]: JSON path where the signature is located
* `type` [string]: signature or timestamp
	* Enum: [signature]
* `status` [object]: validation status for each signature and timestamp included in a signature.
* `time` [string]: date & time from the verification
* `timestamps` [array]: timestamps inluded in a signature. See <a href="#section/JSON-validation-response/#Timestamp-validation-structure">timestamp validation structure</a>
* `certificate_refs` [object]: JSON path where the certificates used for signing are located, ordered by the version number
* `certificates` [array]: list of all certificates used
* `signing_certificate` [string]: certificate used in this signature
* `index` [number]: index of this signature in the list of all file signature, starting from 0
* `name` [string]: signer's name
* `reason` [string]: reason of the signature
* `location` [string]: location where the signature is performed
* `level` [string]:  electronic signature level under the European eIDAS regulation
	* Enum: [PADES-BASIC, PADES-T, PADES-LT, XAdES-BASIC, XAdES-T, XAdES-XL, CAdES-BASIC, CAdES-T]


## Timestamp validation structure

* `data` [string]: JSON path where the signature is located
* `type` [string]: signature or timestamp
	* Enum: [timestamp]
* `status` [object]: validation status for each signature and timestamp included in a signature.
* `time` [string]: date & time of the timestamp
* `tst_info` [string]: timestamp information
* `certificate_refs` [object]: JSON path where the certificates used for signing are located, ordered by the version number
* `certificates` [array]: list of all certificates used
* `signing_certificate` [string]: certificate used in this signature
* `index` [number] : index of this signature in the list of all file signature, starting from 0


## Signature structure

* `version` [number]: JSON path where the signature is located
* `digest_algorithm_identifiers` [array]:
* `signatures` [array]: contains information about signature algorithm used and the digests
* `certificates` [array]: contains all the information from the signer certificates
* `crls` [array]: contains all the information from the CRLs
* `ocsps` [array]: contains all the information from the OCSP certificates


# Postman collection

A postman collection is available as a support for a quick start.<br>
It is only required to edit `host`variable in Postman environment with the IP or domain of VOL Optimizer.

<a href="https://cdn.bit4id.com/es/uanataca/public/vol/Uanataca_VOL_Postman.zip"> VOL Postman collection download</a>

<div id="APIReference" style="padding-top: 60px;"><h1>API Reference<h1></div>
