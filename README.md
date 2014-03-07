# Hale - Hypertext Application Language Extension

## Abstract
This document specifies a proper extension to the 
[Hypertext Application Language (HAL)](http://tools.ietf.org/html/draft-kelly-json-hal-06).

As a proper extension, every HAL document is intended to be a proper Hale document. Hale provides two 
primary extensions to HAL. Specifically it adds a new reserved document keyword called _meta that defines document
*Reference Objects* and extends the properties of HAL *Link Object*s. 

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
 
The base HAL media-type is purposefully light and requires referencing human-readable documentation or content 
negotiation for machine-readable external service descriptor documents (e.g. WADL, RAML, etc.) to understand link 
template values and their constraints, form-related attributes and their constraints, and to determine associated 
uniform interface methods of the protocol expected for the link.

Instead of forcing APIs to externalize service descriptors that lend themselves to tight-coupling or M2M clients having
to understand any arbitrary number of possible machine-readable, service descriptor formats, Hale inherently 
includes features so that a M2M client only needs to understand the Hale media-type to late-bind and interact with an 
API that implements these optional features. 

Hale can be applied to many different domains, and imposes the minimal amount of structure necessary to cover the key 
requirements of a hypermedia API.

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and 
"OPTIONAL" in this document are to be interpreted as described in [RFC2119].

## Overview
The following extensions to HAL are detailed in the following sections:

(1) Extends HAL *Resource Objects* with a new reserved property "_meta" that defines Hale *Reference Objects*.

(2) Introduces Hale *Reference Objects* that include a single reserved property "_ref" for detailing reference
metadata associated with a *Resource Object*.

(3) Extends HAL *Link Objects* to inherit from Hale *Reference Objects* and to include the new properties "method", 
"data", "enctype", and "target".

(4) Introduces Hale *Data Objects* that specify metadata and constraints associated with values that can be set by
clients for URI template variables and request body attributes. 

Example:
```json
{
  "_meta": {
    "lookup": {
       "send_info": { "options": [ "yes", "no", "maybe" ], "in": true }
    },
    "edit_form": {
      "method": "PUT",
      "enctype": "application/json",
      "data": {
        "name": { "type": "string", "required": true },
        "user_id": { "scope": "href" },
        "_ref": ["lookup"]
      }
    }
  },
  "_links": {
    "self": { "href": "..." },
    "search": { 
      "href": ".../{?send_info}",
      "templated": true,
      "method": "GET",
      "data": { "_ref": ["lookup"] }
    },
    "agent": { "href": "/agent/1", "method": "GET", "_ref": [true] },
    "customer": [ 
      { "href": "/customer/1", "method": "GET" },
      { "href": "/customer/2", "method": "GET" }
    ]
  },
  "_embedded": {
    "customer": [
      {
        "_links": {
          "self": { "href": "/customer/1", "method": "GET" },
          "edit": { "href": ".../{?user_id}", "_ref": [ "edit_form" ] }
        },
        "name": "Tom",
        "send_info": "yes"
      },
      {
        "_links": {
          "self": { "href": "/customer/2", "method": "GET" },
          "edit": { "href": ".../{?user_id}", "_ref": [ "edit_form" ] }
        },
        "name": "Harry",
        "send_info": "no"
      }
    ]   
  }
}

```

would be interpreted by a client as:

```json
{
  "_meta": {
    "lookup": {
      "send_info": { "options": [ "yes", "no", "maybe" ], "in": true }
    },
    "edit_form": {
      "method": "PUT",
      "enctype": "application/json",
      "data": {
        "name": { "type": "string", "required": true },
        "send_info": { "options": [ "yes", "no", "maybe" ], "in": true },
        "user_id": { "scope": "href" }
      }
    }
  },
  "_links": {
    "self": { "href": "..." },
    "search": { 
      "href": ".../{?send_info}",
      "templated": true,
      "method": "GET",
      "data": { 
        "send_info": { "options": [ "yes", "no", "maybe" ], "in": true }
      }
    },
    "agent": { "href": "/agent/1", "method": "GET" },
    "customer": [ 
      { "href": "/customer/1", "method": "GET" },
      { "href": "/customer/2", "method": "GET" }
    ]
  },
  "_embedded": {
    "agent": {
      "_links": {
        "self": { "href": "/agent/1", "method": "GET" }
      },
      "name": "Mike"
    },
    "customer": [
      {
        "_links": {
          "self": { "href": "/customer/1", "method": "GET" },
          "edit": { 
            "href": ".../{?user_id}",
            "method": "PUT",
            "enctype": "application/json",
            "data": {
              "name": { "type": "string", "required": true },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true },
              "user_id": { "scope": "href" }
            }
          }
        },
        "name": "Tom",
        "send_info": "yes"
      },
      {
        "_links": {
          "self": { "href": "/customer/2", "method": "GET" },
          "edit": { 
            "href": ".../{?user_id}",
            "method": "PUT",
            "enctype": "application/json",
            "data": {
              "name": { "type": "string", "required": true },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true },
              "user_id": { "scope": "href" }
            }
          }
        },
        "name": "Harry",
        "send_info": "no"
      }
    ]   
  }
}
```

## Resource Objects
Hale adds an additional reserved property to a HAL *Resource Object*:

"_meta": contains reference metadata about the resource.

### Reserved Properties

#### _meta
The reserved "_meta" property is OPTIONAL.

It is an object whose properties provide information about the resource or resource attributes.  The values must be a 
valid JSON object.

Defining a "_meta" attribute automatically defines a *Reference Object* for the related *Resource Object*.

## Reference Objects
Hale introduces *Reference Objects* that represent arbitrary JSON documents which MAY be referenced by other 
JSON objects inside a Hale document. All JSON objects in a *Reference Object*are themselves *Reference Objects*. The 
primary purpose of *Reference Objects* is to simply, yet flexibly, provide a way to DRY documents and extend
other Hale objects in meaningful and useful ways at runtime.

It has one reserved property:

"_ref": contains an ordered array of directives about de-referencing related objects and resources.

### Reserved Properties

#### _ref
The reserved "_ref" property is OPTIONAL.

Valid values are an array of:

(1) strings corresponding to properties of a particular *Reference Object* in a *Resource Object* "_meta" 
tag.

(2) *Link Objects* that should be de-referenced and it's attributes appended to the attributes
of the associated *Reference Object*

(3) or a boolean, which when a property of a Link Object indicates the resource should be immediately de-referenced.

A client SHOULD resolve all "_ref" declarations when encountered in a document according to the order of the array. 

A client MUST resolve a "\_ref" chain hierarchically starting at the top-level in the nearest "\_meta" tag of the 
current *Resource Object*. Given the recursive structure of HAL, if the particular value is not discovered in the 
current *Resource Object*, the client SHOULD look in a *Resource Object* embedding the current *Resource Object*, if
any.

The inheritance hierarchy for common *Reference Object* attributes is the value of the de-referencing *Reference Object* 
supersedes those in the objects referenced.  

###### strings
A string indicates the name of a Reference object property in a "_meta" tag in the current *Resource Object* or another
*Resource Object* embedding the current resource object.

Example:

```json
{
  "_meta": {
    "data": {
      "options": [0, 1, 2],
      "value": 0
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
          "_ref": ["something_else"]
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
    },
    "data1": {
      "options": [0, 1, 2],
      "value": 1
    },
    "something": {
      "max": 1,
      "value": 2
    },
    "something_else": {
      "options": [0, 1, 2],
      "max": 1,
      "value": 2
    }
  },
  "_embedded": {
    "_meta": {
      "embedded_something": {
        "options": [0, 1, 2],
        "max": 1,
        "value": 2
      }
    }
  }
}
```

####  Link Objects
If a value is a Hale *Link Object*, the client SHOULD dereference the associated resource and content-negotiate 
media-types it understands, if the`type` attribute is not specified in the *Link Object*.

Unless a "target" property is specified in the *Link Object*, the client SHOULD completely fill in any attributes of the 
*Reference Object *with those of the return ed resource with the caveat that in the case of like attributes, 
the attribute of the *Resource Object* supersedes that of the de-referenced resource. See *Link Object* 
["target"](/README.md#target) property for more information.

Example:

```json
{
  "_meta": {
    "explosion": {
      "occupation": "swamp thing",
      "_ref": { "rel": "person", "href": "...", "method": "GET", "type": "application/json" }
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
  "occupation": "Scientist"
}
```

```json
{
  "_meta": {
    "explosion": {
      "name": "Alex Olsen",
      "occupation": "Swamp Thing"
    }
  }
}
```

##### boolean
A boolean value is only applicable to a *Link Object* in a "\_links" tag. If `true` it indicates that the link SHOULD be
immediately de-referenced by a client and appended to the "\_embedded" property using the same name as the "rel" of the
*Link Object*.

## *Data Objects*
Hale *Data Objects* inherit from Hale *Reference Objects* and specify metadata and constraints associated with values 
that can be set by a client in exercising a link. A *Data Object* has the following properties:

### Metadata Properties

#### description
The "description" property is OPTIONAL.

Specifies a Human Readable Description.

#### prompt
TODO: Do we want this?

#### type
The "type" property is OPTIONAL.

Specifies the expected type. Allowed values are "string", "number", or "object".

#### scope
The "scope" property is OPTIONAL.

Specifies the scope of the input value. Allowed values are "href" or "body". 

Specifying "href" indicates the object only applies to URI template variables with the same name. The value "body" 
indicates the object only applies to generating a body object to send with the request. 

If unspecified, the object only applies to the request body.

#### data
Allows recursively defining data properties of objects if a particular input is structured.

TODO: Is this necessary or should a service just 422 and send information to correct the issue? Do what extent can
profiles help this.

#### profile
TODO: would this make sense.


#### value
The "value" property is OPTIONAL.

Specifies the current or default value

### Constraint Properties

#### options
The "options" property is OPTIONAL.

If options is specified it MUST be a list. Options specifies possible for the associated data. The list can either be
an array of "strings" or an array of JSON objects keyed by the value to be used (this may need to be a Reference
Object) to specify more about what property to use).

#### in
The "in" property is OPTIONAL.

In is a boolean. If "in" is specified the Constraint Object must specify "options".  The constraint "in" 
specifies that the value of a request parameter or attribute MUST be within the specified set of "options" in 
order to fulfill the relationship.

#### min
The "min" property is OPTIONAL.

The constraint 'min' specifies that the value of a request parameter is below a certain value. If the value is a 
string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

#### max
The "max" property is OPTIONAL.

The constraint 'max' specifies that the value of a request parameter is above a certain value. If the value is a 
string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

#### maxlength
The "maxlength" property is OPTIONAL.

Specifies the maximum length of a string, the maximum number of items in a list, or the maximum number of digits in a 
number.

#### pattern
The "pattern" property is OPTIONAL.

Pattern specifies the PCRE regular expression that a string must conform to.

#### multi
The "multi" property is OPTIONAL.

Multi is a boolean that specifies whether or not more than one of an item is allowed.  (i.e. ?name=foo&name=bar)

#### required
The "required" property is OPTIONAL.

This object must be a boolean.  When a Constraint Object specifies the Required this attribute must be included in the 
request to fulfill the relationship.

#### Constraint Extensions

Other attributes are considered constraint extensions. It may be an object or a reference. If it is a reference it must 
refer to something which conforms. Anything under a constrain extension MUST specify a profile tag giving a 
link that describes the constraint. The other attributes must match the tags specified in the profile.

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
            "href": "http://deployment.example.org/drds",
            "method": "POST",
            "data": {
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
            "templated": true,
            "data": {
              "search_term": {
                "maxlength": 50,
                "scope": "href"
              }
            }
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
}
```

## *Link Object*s
Hale extends *Link Object*s to inherit from Hale *Reference Objects* and introduces the following new properties:

### method
The "method" property is OPTIONAL.

It specifies the uniform-interface method (e.g. HTTP.GET) that fulfills the specified relationship.

### data
The "data" property is OPTIONAL.

Keyed by input name that matches a URI template variable or a body property, it specifies the Input Objects defining 
the allowed vales in exercising a link. Valid values are a single Input Object or an array of Input Objects for the
rare but possible case where a naming collision exists between a URI template variable and a body property.

Example:

```json
{
  "_links": {
    "update": {
      "href": ".../{?search}",
      "data": {
        "name": { "value": "Mike" },
        "search" [   
          { "value": "yes", "scope": "body", "options": ["yes", "no"] }, 
          { "scope": "href" }
        ]
      }
    }
  }
}
```

### enctype
The "enctype" property is OPTIONAL.

Specifies the media type that should be used to make the request.

MUST be a list or a string.  If it is a string the server only accepts requests of that media-type.
If it is a list, the server will accept any of the media-types specified.

Defaults to application/x-www-form-urlencoded.

### target

The reserved "target" property is OPTIONAL.

"target" specifies that the information the subset of associated resource to return.  If the "type" property is 
JSON then "target" should specify an JSONPATH [http://goessner.net/articles/JsonPath/], if it is XML it 
should specify an XPATH [RFC 5261]. 
 
The "target" property allows *Reference Objects* to be created from remote resources for use in generating "options" for 
input attributes.

## Authors

Shea Valentine and Mark W. Foster
