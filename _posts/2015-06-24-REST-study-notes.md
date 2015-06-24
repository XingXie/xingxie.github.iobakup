---
layout: post
title:  "Restful API Study Notes"
date:   2015-06-24
categories: REST
excerpt: Restful API
---

* content
{:toc}

## upload multiple files
~~~ java
   MultiPart multiPartEntity = new FormDataMultiPart().field("statements", stmts.toString());
if (uploadFiles != null && uploadFiles.size() > 0) {

for (String f : uploadFiles) {
 FileDataBodyPart filePart = new FileDataBodyPart("data", new File(f));
 multiPartEntity = multiPartEntity.bodyPart(filePart);
    }


   }
WebTarget target = server.getTarget();
   resultStr = target.path("statements")
     .request()
     // MediaType.APPLICATION_JSON_TYPE)
     .header("Wings-Tracking-Authorization-Key", trackerAuthKey)
     .header("Wings-Tracking-API-Key", server.getTrackerApiKey())
     .post(Entity.entity(multiPartEntity, multiPartEntity.getMediaType()), String.class);
   // logger.info("valid statements result string: " + resultStr);
   jsonNode = om.readValue(resultStr, JsonNode.class);
~~~

REST is offered not as a procotol, but as an architectural paradigm. However, in reality we are pretty much talking about HTTP of which REST is an abstraction. The core aspects of the architecture are
1. resource identifiers (i.e. URIs);

2. different possible representations of resources, or internet media types (e.g. application/json);

3. CRUD operations support for resources like the HTTP methods  GET, PUT, POST and DEL.

// HTTP Negotiation
Internationalization can be negotiated to
* CONTENT-LANGUAGE:whatlanguageistherequestbody
* ACCEPT-LANGUAGE:whatlanguageisdesiredbyclient

GET /customers/33323
ACCEPT: application/xml
ACCEPT-LANGUAGE: en_US

￼JAX-RS Annotations
@Path
* Defines URI mappings and templates

@Produces, @Consumes
* What MIME types does the resource produce and consume

@GET, @POST, @DELETE, @PUT, @HEAD

* Identifies which HTTP method the Java method is interested in
