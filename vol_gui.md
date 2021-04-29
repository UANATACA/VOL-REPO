# What it is

<div style="text-align: justify">
Uantaca VOL is a high performance, enterprise-grade solution for bulk electronic signature validation. VOL is a turnkey system that can be easily integrated with any business application thanks to its high-level web API based on the http RESTful paradigm.
</div>

# How it works
<div style="text-align: justify">
The service is given with VOL Optimizer that is a server system exposing http RESTful APIs by means of which, business applications are enabled to require the electronic signature validation of files batch.
<br></br>
VOL Optimizer performs the most computationally expensive workload of the validation process, thus reducing the data traffic on the local network. The signed documents to be validated are processed in the customer business layer and are not send to Uanataca Services or to the Certification Authorities.
<br></br>
</div>
The system performs the validation on all the signatures of a document, including the time stamps. The validation consists on:

- verify that the certificate is issued by a Trusted Service Provider (TSP)
- verify that the certificate was active at the time of signing. It means that the certificate was not expired or revoked at this moment
- verify the integrity of the document, checking that no changes are made to the document after the signature 


The verification of the digital certificate is carried out through OCSP and CRL queries to the corresponding TSP issuer.
<br></br>


![img](https://raw.githubusercontent.com/UANATACA/VOL-REPO/main/img/vol_hiw.png?token=ATF574TSYH5QNMNK6K5I7P3ALXINS)


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


![img](https://raw.githubusercontent.com/UANATACA/ONESHOT-REPO/main/img/vol-docker1.png?token=ATF574UUBXJRM4MJH7PE5JTALXEUO) 

</br>

> STEP 2: Extract and copy VOL Optimizer zip content to the server.

Extract all `vol_optimizer_docker.zip` content in a local folder.

Move VOL Optimizer folder to the path **/opt** in the server.

The outcome should look like this:

![img](https://raw.githubusercontent.com/UANATACA/ONESHOT-REPO/main/img/vol-docker2.png?token=ATF574R6PU4TUA3NT35UXRDALXFBI)

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

Run:

	docker-compose up -d

</br>

> STEP 5: Web settings.

See <a href="#section/Configuration/Web-settings">web settings</a> configuration section.

</br>


## Web settings

<div style="text-align: justify">
Uanataca VOL exposes a custom website to validate documents.
</div>


**Requirements:**

- Billing credentials for Uanataca production environment.
- SSL Certificate (.cer) and key (.key) files.


> STEP 1: SSL Certificate.

Run the following commands:

	cd /home/bit4id/vol-volumes/vol-web-storage/

Copy SSL certificate files

</br>

> STEP 2: Billing credentials.

Copy `vol-gui-passwd.tar` file to the path **/home/bit4id/** in the server.
Extract the content:

	cd /home/bit4id/
	tar -xvf vol-gui-passwd.tar.gz

Run the script to set Billing credentials:

	./passwd

</br>

> STEP 3: Launch the service.

Run:

	cd /opt/vol_optimizer/
	systemctl start docker
	docker-compose start

Check service status:

	docker-compose ps

</br>



# Endpoint URLs

The URL where is exposed the API is composed as follows:

	{host}/api/{resource}/

> host

The {host} is the IP or domain from the VOL Optimizer.

> resource

It is the name of the resource of our interest.

Each resource can also have path parameters and sub-resources that are defined in the <a href="#APIReference">API Reference</a> below:

This is an example of endpoint:

	https://vol.uanataca.com/api/admin/certificates/

# Authentication

VOL service requires authentication with the provided **Billing credentials**.
 
Each endpoint requires an authentication header before performing the requested action.
The header must containt the type (Basic) and a Base64 string that contains the Billing credentials in the format **username:password** in Base64.

	Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

This is an example with cURL:

	1 | curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" -X GET https://vol.uanataca.com/api/documents/06618cc5d920cb09/content


# Validation response structure



- Report
- Errors
- Data
- Messages
- Validation

donde los atributos son:


* **`report`**

    * **`now`**

        date & time from the verification

    * **`type`**

        document type

    * **`document_uri`**

        ruta para recuperar el documento a través del interfaz web

    * **`signed_data`**

        objeto `signed_data` ya descirto en la 
        [estructura de firma](#struttura-di-firma)


* **`errors`**

    Lista de los errores encontrados durante la validación. Cada uno contiene:

    * **`where`**

        identificativo de donde ha sido encontrado el error

    * **`what`**

        identificativo del mensaje de error, [verifica `mensajes`](#messages)


* **`data`**

    Contiene información relativa a la carga del archivo en el sistema:

    * **`upload_time`**

        date & time from the document upload

    * **`outformat`**

        formato de la respuesta

    * **`filename`**

        nombre del archivo cargado

    * **`headers`**

        header utilizados en la solicitud HTTP de carga (método POST)

    * **`document_hash_type`**

        algoritmo de hash utilizado

    * **`template`**

        tipo de plantilla solicitada para la validación

    * **`report_uri`**

        ruta para recuperar el informe de validación a través del interfaz web


    * **`verification_date`**

        fecha en la que se hace la valicación (opcional en la solicitud)

    * **`documentid`**

        el **document-id**, está representado por las primeras diecieséis 
		cifras hexadecimales del hash

    * **`document_hash`**

        hash completo del documento según el algoritmo utilizado


* **`messages`**

    Contiene códigos de país (*ISO 3166-1 alpha-2*).

    La propiedad `what` de los errores se refiere a las claves de este objeto.


* **`validation`**

    Contiene los resultados de las validaciones sobre cada atributo.

    * **`profile`**

        tipo de perfil utilizado en la validación

    * **`signatures`**

        contiene los resultados de la validación de las firmas

        * **`data`**

            la ruta JSON donde se encuentra la firma

        * **`type`**

            tipo de firma

        * **`status`**

            éxito de la validación

        * **`time`**

            fecha y hora de la validación

        * **`timestamps`**

            sellos de tiempo (opcional)

        * **`certificate_refs`**

            ruta JSON donde se encuentran los certificados utilizados para la firma,
            ordenados según el número de versión

        * **`certificates`**

            lista de todos los certificados utilizados

        * **`signing_certificate`**

            certificaco utilizado para esta firma

        * **`index`**

            índice de esta firma dentro de la lista de todas las firmas
            presentes en el documento (la numeración inicia desde cero)

        * **`countersignatures`**

            contrafirmas (si las hubiera)

        * **`level`**

            perfil de la firma (según aquellos previstos `eIDAS`)

    * **`timestamps`**

        contiene los resultados de la validación de los sellos de tiempo
		(opcional)

A continuación se muestra un ejemplo de estructura de validación:

```json
{
  "report": {
    "now": "2017-12-20T11:46:36.862950",
    "type": "CAdES",
    "document_uri": "/contents?documentid=c1cab389ac304222",
    "signed_data": {…}
  },
  "errors": [
    {
      "where": "x509/extensions/qcStatements/QcPDS",
      "what": "INVALID_ENCODING"
    }
  ],
  "data": {
    "upload_time": "2017-12-20T11:46:36.320074",
    "outformat": "json",
    "filename": "cades-text.p7m",
    "headers": {
      "CONTENT-LENGTH": "3402",
      "ACCEPT": "*/*",
      "USER-AGENT": "curl/7.47.0",
      "HOST": "docky",
      "EXPECT": "100-continue",
      "CONTENT-TYPE": "multipart/form-data; boundary=------------------------f699eebb362e5038",
      "AUTHORIZATION": "token bef3ba974d7c8e183a70d7264ba40890"
    },
    "document_hash_type": "sha256",
    "template": "report",
    "report_uri": "/reports?documentid=c1cab389ac304222",
    "verification_date": null,
    "documentid": "c1cab389ac304222",
    "document_hash": "c1cab389ac30422251bd723f6b508930251f2e8b0e0ab62f4ffed256f2c68ef1"
  },
  "messages": {
    "it": {
      "INVALID_CONTENT_TYPE_ATTRIBUTE": "Atributo de la firma no conforme (contentType)",
      "MULTIPLE_SIGNING_TIME_ATTRIBUTE": "Fecha y hora de la firma declarada más veces
      (signingTime)",
      …
      "SIGNING_CERTIFICATE_REF_FOUND": "Referencia al certificado de firma válido"
    }
  },
  "validation": {
    "profile": "CAdES",
    "signatures": […],
    "timestamps": []
  }
}
```

# Postman collection

A postman collection is available as a support for a quick start.<br>
It is only required to edit `host`variable in Postman environment with the IP or domain of VOL Optimizer.

<a href="https://cdn.bit4id.com/es/uanataca/public/VOL/Uanataca_VOL_Postman.zip"> VOL Postman collection download</a>

<div id="APIReference" style="padding-top: 60px;"><h1>API Reference<h1></div>
