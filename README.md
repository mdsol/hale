# Hale - Hypertext Application Language Extension

## Abstract
This document specifies a proper extension to HAL as defined at
http://tools.ietf.org/html/draft-kelly-json-hal-06

As a proper extension every HAL document is intended to be a proper Hale document and vice-versa. Hale provides two 
primary extensions to HAL. Specifically it adds a new reserved document keyword called _meta that defines a document
Reference Object and extends the properties of HAL Link Objects. 

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
their constraints, form attributes and their constraints for particular links, and to determine associated 
uniform interface methods of the protocol expected for the link.

Instead of forcing APIs to externalize service descriptors that lend themselves to tight-coupling or M2M clients to 
understand any arbitrary number of possible machine-readable, external, service descriptor formats, Hale inherently 
includes features so that a M2M client only needs to understand the Hale media-type to late-bind and interact with an 
API that implements these optional features.

Hale can be applied to many different domains, and imposes the minimal amount of structure necessary to cover the key 
requirements of a hypermedia API.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [RFC2119].

## Resource Objects
Hale adds an additional reserved property to a HAL Resource Object:

(1) "_meta": contains reference metadata about the resource.

### Reserved Properties

#### _meta
The reserved "_meta" property is OPTIONAL.

It is an object whose properties provide information about the resource or resource attributes.  The values must be a 
valid JSON object.

Defining a _meta attribute automatically defines a document *Reference Object*.

## Reference Objects
Hale introduces Reference Objects that represents an arbitrary JSON document which MAY be referenced by other 
JSON objects inside a Hale document.

It has two reserved properties:

(1) "_ref": contains an array of Reference Object properties chain-ordered for inheriting attributes from other JSON 
objects.
(2) "_src": contains a HAL link object referencing a remote resource.

### Reserved Properties

#### _ref
The reserved "_ref" property is OPTIONAL.

It is an array of strings corresponding to properties of particular Reference Object in a document. A client SHOULD 
resolve a "_ref" chain when encountered in a Reference Object in a document. 

A client MUST resolve a "_ref" chain hierarchically starting in the nearest "_meta" tag of the current Resource Object. 
Given the recursive structure of HAL, if the particular value is not discovered in the current Resource Object, the 
client SHOULD look in a Resource Object embedding the current Resource Object.

The inheritance hierarchy for common Reference Object attributes is the value of the current Reference Object supersedes
those in the objects specified.

The client should first evaluate any classes that exist in the top level of the document and apply the class 
attribute to any sub attribute anywhere in the document.  If _meta has been defined in an embedded resource, 
the _meta classes only apply to the attributes within that resource.  


Example:

```json
{
  "_meta": {
    "data": {
      "options": [0, 1, 2],
      "value": 0
      }
    },
    "data1": {
      "_ref": ["data"],
      "value": 1
    },
    "something": {
      "max": 1,
      "value": 2
    },
    "something_else": {
      "_ref": ["data1", "something"]
    },
    "_embedded" : {
      "_meta": {
        "embedded_something": {
          "_ref": ["something"]
          "value": 4
        }
      }
    }
  }
}
```

would be interpreted as:

```json
{
  "_meta": {
    "data": {
      "options": [0, 1, 2],
      "value": 0
      }
    },
    "data1": {
      "options": [0, 1, 2],
      "value": 1
    },
    "something": {
      "max": 1,
      "value": 2
    }
    "something_else": {
      "options": [0, 1, 2],
      "max": 1
      "value": 2
    }
  },
  "_embedded" : {
     "_meta": {
       "embedded_something": {
         "max": 1
         "value": 4
       }
     }
   }
}
```

####  _src
The reserved "_src" property is OPTIONAL.

Its value is a HAL Link Object.

The client SHOULD dereference the associated resource and conneg media-types it understands, if not specified in the 
Link Object. 

Unless a "_target" property is specified, the client SHOULD fill in any attributes of the Reference Object with those 
of the resource with the caveat that in the case of like attributes, the attribute of the Resource Object supersedes 
that of the de-referenced resource.

Example:

```json
{
  "_meta": {
    "data": {
      "occupation": "lurking",
      "_src": { "rel": "thing", "href": "...", "type": "application/json" }
    }
  }
}
```

would be interpreted as:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 273

{
  "name": "Alex Olsen",
  "occupation": "scientist"
}
```

```json
{
  "_meta": {
    "data": {
      "name": "Alex Olsen",
      "occupation": "lurking"
    }
  }
}
```

## Link Objects
A Link Object represents a relationship from the containing resource to a URI, and the information necessary to fulfill 
that relationship.
It has the following properties:

### method
The "method" property is OPTIONAL.

It specifies the uniform-interface method (e.g. HTTP.GET) that fulfills the specified relationship.

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

### _target

The reserved "_target" property is OPTIONAL.

_target specifies that the information from a _source target can be found in a specified location.  If the media 
format is JSON then target should specify an JSONPATH [http://goessner.net/articles/JsonPath/], if it is XML it 
should specify an XPATH [RFC 5261].  


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
