# Introduction

## What it is

The VOL (On-Line Verifier) allows the verification of digital signed documents.
<br></br>

![img](https://i.ibb.co/pXWHTYD/vol-wkf.png)

## Authentication

Each endpoint requires an authentication header before performing the requested action.

The header must containt the type (Basic) and a Base64 string that contains the username and password in the format **username:password**.

So the header will be something like:

	Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

where **dXNlcm5hbWU6cGFzc3dvcmQ=** is the Base64 encoded string of **username:password**

### Authentication header sent with curl:

	1 | curl -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=" -X GET https://vol.uanataca.loc/api/documents/06618cc5d920cb09/content

## URLs

The URL where are exposed the API is composed as follows:

	https://{volhost}/api/{resource}/

The {volhost} changes according to the environment:

- vol.access.bit4id.org:13035 if the environment is test
- vol.uanataca.com if the environment is production
- and **{resource}** is the name of the resource of our interest.

Each resource can also have path parameters and sub-resources:

	https://{volhost}/api/{resource}/{param}/{sub-resources}/

</br>

	https://vol.uanataca.com/api/admin/certificates/

# Watch on Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/tUGLf5y1dEA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
