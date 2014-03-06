# Hale - Hypertext Application Language Extension
====
## Abstract
This document specifies a proper extension to HAL as defined at
http://tools.ietf.org/html/draft-kelly-json-hal-06

As a proper extension every HAL document is intended to be a proper Hale document and vice-versa. Hale provides two 
primary extensions to HAL. Specifically it extends the properties of Link Objects and adds a new reserved document 
keyword called _meta.

## Introduction

There is an emergence of non-HTML HTTP applications ("Web APIs") which use hyperlinks to direct clients around their 
resources.

The JSON Hypertext Application Language Extension (Hale) is a standard which establishes conventions for expressing 
hypermedia documents with JSON [RFC4627].

Hale is a general purpose media type with which Web APIs can be developed. Clients of these APIs can interact with 
services by reference to their relation type and use the Hale document to fulfill those relationships in order to 
progress through the application.  

Hale's conventions result in a uniform interface for serving and consuming hypermedia, enabling the creation of 
general-purpose libraries that can be re-used on any API utilizing Hale.

The primary design goals of Hale are completeness, generality, simplicity and machine-to-machine (M2M) friendliness, the 
later being a primary motivator for the extension of HAL. 
 
The HAL media-type is purposefully light and requires referencing human-readable documentation or CONNEG with 
machine-readable external service descriptor documents (e.g. WADL, RAML, etc.) to understand href template values and 
their constraints, body attributes and their constraints for particular links, and to determine associated 
uniform interface methods of the protocol expected for the link.

Instead of forcing M2M clients to understand any arbitrary number of possible machine-readable, external, service 
descriptor formats, Hale inherently includes features so that an M2M client only needs to understand the Hale 
media-type to interact with an API that implements these optional features.

Hale can be applied to many different domains, and imposes the minimal amount of structure necessary to cover the key 
requirements of a hypermedia API.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [RFC2119].

## Hale Documents
A Hale Document uses the format described in [RFC4627] and has the media type "application/vnd.hale+json".

Its root object MUST be a Resource Object.

For example:
```
   GET /orders/523 HTTP/1.1
   Host: example.org
   Accept: application/vnd.hale+json

   HTTP/1.1 200 OK
   Content-Type: application/vnd.hale+json
```
```json
   {
     "_links": {
       "self": { "href": "/orders/523" },
       "warehouse": { "href": "/warehouse/56" },
       "invoice": { "href": "/invoices/873" },
       "process": { "href": "/orders/523/process" }
     },
     "currency": "USD",
     "status": "shipped",
     "total": 10.20
   }
```

Here, we have a HAL document representing an order resource with the URI "/orders/523".  It has "warehouse" and 
"invoice" links, and its own state in the form of "currency", "status", and "total" properties.

_(to do: expand)_

## Resource Objects
A Resource Object represents a resource.

It has three reserved properties:

1. "_meta": contains information about the resource or resource elements which are not themselves resource attributes.
2. "_links": contains information about relationships with other resources
3. "_embedded": contains an embedded resource which must be a valid Hale document

All other properties MUST be valid JSON, and represent the current state of the resource.

### Reserved Properties

### _meta
The reserved "_meta" property is OPTIONAL.

It is an object whose properties provide information about the resource or resource attributes.  
The values must be a valid JSON object.
Defining a _meta attribute automatically defines a document *Class* Object.

### _links
The reserved "_links" property is OPTIONAL.

It is an object whose property names are link relation types (as defined by [RFC5988]) and values are either a Link Object or an array of Link Objects.  The subject resource of these links is the Resource Object of which the containing "_links" object is a property.

### _embedded
The reserved "_embedded" property is OPTIONAL

It is an object whose property names are link relation types (as defined by [RFC5988]) and values are either a Hale Object or an array of Hale Objects.

Embedded Resources MAY be a full, partial, or inconsistent version of the representation served from the target URI.

## Class Objects
A class object represents an arbitrary json document which MAY be referenced by other JSON keys inside a Hale document.
A Class Objects reserves the properties _source and _target.

### Class References
When a _meta attribute defines a Class Object, all attributes defined for that class apply to any attribute that references that class.  A class reference starts with "className." and the what follows the "." is treated as that attribute name.

For example

```json
{
  "_meta": {
    "data": {
      "options": [0,1,2],
      "value": 2
      }
    },
  "data.something": {
    "max": 1,
    "value": 0,
  }
}
```

would be interpreted as

```json
{
  "_meta": {
    "data": {
      "options": [0,1,2],
      "value": 2
      }
    },
  "something": {
    "max": 1,
    "value": 0,
    "options": [0,1,2]
  }
}
```

The client should first evaluate any classes that exist in the top level of the document and apply the class attribute to any sub attribute anywhere in the document.  If _meta has been defined in an embedded resource, the _meta classes only apply to the attributes within that resource.  

A class object also has the following properties:


### Multiple Classes
An object may reference multiple classes.  When evaluating these attributes first apply the top level class, then apply the next class, and so forth.

E.g.
```json
{
  "_meta": {
    "data": {
      "options": [0,1,2]
      "value": 2
      },
    "data2": {
      "options": [1,2,3]
      }
    },
  "data.data2.something": {
    "max": 1,
    "value": 0,
  }
}
```

would be interpreted as

```json
{
  "_meta": {
    "data": {
      "options": [0,1,2]
      "value": 2
      },
    "data2": {
      "widget": true,
      "options": [1,2,3]
      }
    },
  "something": {
    "max": 1,
    "value": 0,
    "options": [1,2,3],
    "widget": true
  }
}
```

### Mixing Classes and Curies
Classes have a higher operator precedence than curies.  A client should apply all class dereferences before dereferencing curies.

### _source
_source tells the client that it needs to find more information from a remote resource.  The client is instructed to dereference that resource and fill in any class attributes with those specified.

### _target
_target specifies that the information from a _source target can be found in a specified location.  If the media format is JSON then target should specify an JSONPATH [http://goessner.net/articles/JsonPath/], if it is XML it should specify an XPATH [RFC 5261].  


## Link Objects
A Link Object represents a relationship from the containing resource to a URI, and the information necessary to fulfill that relationship.
It has the following properties:

### href
The "href" property is REQUIRED.

Its value is either a URI [RFC3986] or a URI Template [RFC6570].

If the value is a URI Template then the Link Object SHOULD have a "templated" attribute whose value is true.

### templated
The "templated" property is OPTIONAL.

Its value is boolean and SHOULD be true when the Link Object's "href" property is a URI Template.

Its value SHOULD be considered false if it is undefined, or any other value than true.

### type
The "type" property is OPTIONAL.

Its value is a string used as a hint to indicate the media type expected when dereferencing the target resource.

### deprecation
The "deprecation" property is OPTIONAL.

Its presence indicates that the link is to be deprecated (i.e. removed) at a future date.  Its value is a URL that SHOULD provide further information about the deprecation.

A client SHOULD provide some notification (for example, by logging a warning message) whenever it traverses over a link that has this property.  The notification SHOULD include the deprecation property's value so that a client maintainer can easily find information about the deprecation.

### name
The "name" property is OPTIONAL.

Its value MAY be used as a secondary key for selecting Link Objects which share the same relation type.

### profile
The "profile" property is OPTIONAL.

Its value is a string which is a URI that hints about the profile (as defined by [I-D.wilde-profile-link]) of the target resource.

### title
The "title" property is OPTIONAL.

Its value is a string and is intended for labeling the link with a human-readable identifier (as defined by [RFC5988]).

### hreflang
The "hreflang" property is OPTIONAL.

Its value is a string and is intended for indicating the language of the target resource (as defined by [RFC5988]).

### method
The "method" property is OPTIONAL.

It specifies the uniform-interface method (e.g. HTTP) method that fulfills the specified relationship.

### parameters
The "parameters" property is OPTIONAL.

It specifies Constraint Objects which apply to the URL Template variables. A parameters attribute implies that the specified URL is templated. A parameter attribute implies a constraint on a url parameter and should be included if there is a constraint.

(consider removing this and subbing templated)

### attributes
The "attributes" property is OPTIONAL.

It specifies Constraint Objects which apply to the Request body. Specifying attributes implies that the specified relationship requires a Request Body to be fulfilled.

### enctype
The "enctype" property is OPTIONAL.

May be a list or a string.  If it is a string the server only accepts requests of that mediatype.
If it is a lite, the server will accept any of the mediatypes specified.
Specifies the media type that should be used to make the request.
Defaults to application/x-www-form-urlencoded

## Constraint Objects
A constraint object may specify a Default or Constraint Parameters.  
If it specifies a Default it MUST be either a literal (Number or String), or a List.
If it specifies Constraint Parameters it must be an object.

### Default
A (string or number) specifies that the attribute is of that type, and is of that value currently or by default.
A List specifies multiple values set for the same parameter, equivalent to a "name=John&name=Jane".

### Constraint Parameters
A Constraint Objects has the following reserved properties.  Any other value will be considered a Validator Object.

#### description
The "description" property is OPTIONAL.

Specifies a Human Readable Description.

#### type
The "type" property is OPTIONAL.

Specifies the expected parameter type.

#### options
The "options" property is OPTIONAL.

If options is specified it MUST be a list. Options specifies possible values.

#### value
The "value" property is OPTIONAL.

value - specifies the current or default value

## Validator Objects
A validator specifies a property of a Constraint Object that must be true in order to fulfill a relationship.

The following are all reserved validators.  Any other property will be considered a Validator Extension.

### in
The "in" property is OPTIONAL.

In is a boolean. If "in" is specified the Constraint Object must specify "options".  The constraint "in" specifies that the value of a request parameter or attribute MUST be within the specified set of "options" in order to fulfill the relationship.

### min
The "min" property is OPTIONAL.

The constraint 'min' specifies that the value of a request parameter is below a certain value. If the value is a string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

### max
The "max" property is OPTIONAL.

The constraint 'max' specifies that the value of a request parameter is above a certain value. If the value is a string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

### maxlength
The "maxlength" property is OPTIONAL.

Specifies the maximum length of a string, the maximum number of items in a list, or the maximum number of digits in a number.

### pattern
The "pattern" property is OPTIONAL.

Pattern specifies the PCRE regular expression that a string must conform to.

### multi
The "multi" property is OPTIONAL.

Multi is a boolean that specifies whether or not more than one of an item is allowed.  (i.e. ?name=foo&name=bar)

### required
The "required" property is OPTIONAL.

This object must be a boolean.  When a Constraint Object specifies the Required this attribute must be included in the request to fulfill the relationship.

### constraint extensions

Other attributes are considered constraint extensions. It may be an object or a reference. If it is a reference it must refer to something which conforms. Anything under a constrain extension MUST specify a profile tag giving a link that describes the constraint. The other attributes must match the tags specified in the profile.

## Example Document
```json
{
"_meta": {
   "locations": {
      "options": [
         "location1",
         "location2",
         "location3",
         "location4",
         "location5",
         "location6",
         "location7",
         "location8",
         "location9",
         "location10"
      ]}
   },
"_links": {
        "profile": {
            "href": "http://alps.example.org/DRDs"
        },
        "create": {
            "href": "http://deployment.example.org/drds{?create-drd}",
            "templated": true,
            "method": "POST",
            "attributes": {
               "status": {
                  "required": true
                  },
               "kind": {
                  "options": [
                    "standard",
                    "sentinel"
                    ]
                  "in": true
                  }
               "locations.location": {},
               "form-destroyed": {
                  "type": "Bool"
                  },
               "name" {
                  "required": true,
                  "maxlength": 50
                  }
               "leviathan_uuid": "",
               "email": {
                  "pattern": "^.+@.+$"
                  }
               }
            },
        "search": {
            "href": "http://deployment.example.org/drds{?search_term}",
            "templated": true
        },
        "help": {
            "href": "http://documentation.example.org/Things/DRDs"
        },
        "self": {
            "href": "http://deployment.example.org/drds"
        },
        "type": {
            "href": "http://alps.example.org/DRDs#drds"
        },
        "items": [
            {
                "href": "http://deployment.example.org/drds/0",
                "type": "http://alps.example.org/DRDs#drd"
            },
            {
                "href": "http://deployment.example.org/drds/1",
                "type": "http://alps.example.org/DRDs#drd"
            }
        ]   
```

## Media Type Parameters

### profile

The media type identifier application/vnd.hale+json MAY also include an additional "profile" parameter (as defined by [I-D.wilde-profile-link])

Hale documents that are served with the "profile" parameter still SHOULD include a "profile" relationship belonging to the root resource.

## Recommendations
### Self Link
Each Resource Object SHOULD contain a 'self' link that corresponds with the IANA registered 'self' relation (as defined by [RFC5988]) whose target is the resource's URI.

### Link Relations
Custom link relation types (Extension Relation Types in [RFC5988]) SHOULD be URIs that when dereferenced in a web browser provide relevant documentation, in the form of an HTML page, about the meaning and/or behavior of the target Resource.  This will improve the discoverability of the API.

The CURIE Syntax [W3C.NOTE-curie-20101216] or Document Classes MAY be used for brevity for these URIs.  CURIEs are established within a Hale document via a set of Link Objects with the relation type "curies" on the root Resource Object. Classes are established within a Hale document via the _meta property. 
These links contain a URI Template with the token 'rel', and are named via the "name" property.
```json
   {
     "_links": {
       "self": { "href": "/orders" },
       "curies": [{
         "name": "acme",
         "href": "http://docs.acme.com/relations/{rel}",
         "templated": true
       }],
       "acme:widgets": { "href": "/widgets" }
     }
   }
```

The above demonstrates the relation "http://docs.acme.com/relations/widgets" being abbreviated to "acme:widgets" via CURIE syntax.

### Hypertext Cache Pattern

The "hypertext cache pattern" allows servers to use embedded resources to dynamically reduce the number of requests a client
 makes, improving the efficiency and performance of the application.

Clients MAY be automated for this purpose so that, for any given link relation, they will read from an embedded resource (if present) in preference to traversing a link.

To activate this client behavior for a given link, servers SHOULD add an embedded resource into the representation with the same relation.

Servers SHOULD NOT entirely "swap out" a link for an embedded resource (or vice versa) because client support for this technique is OPTIONAL.

The following examples shows the hypertext cache pattern applied to an "author" link:
Before:
```json
   {
     "_links": {
       "self": { "href": "/books/the-way-of-zen" },
       "author": { "href": "/people/alan-watts" }
     }
   }
```


   After:
```json
   {
     "_links": {
       "self": { "href": "/blog-post" },
       "author": { "href": "/people/alan-watts" }
     },
     "_embedded": {
       "author": {
         "_links": { "self": { "href": "/people/alan-watts" } },
         "name": "Alan Watts",
         "born": "January 6, 1915",
         "died": "November 16, 1973"
       }
     }
   }
```

## Security Considerations
...

## IANA Considerations
...

## Appendix
### Acknowledgements
M. Kelly

### Frequently Asked Questions
