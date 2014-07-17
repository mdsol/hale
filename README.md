# *Hale* - Hypertext Application Language Extension

## Abstract
This document specifies a proper extension to the 
[Hypertext Application Language (HAL)](http://tools.ietf.org/html/draft-kelly-json-hal-06).

As a proper extension, every HAL document is intended to be a proper *Hale* document and vice versa. *Hale* provides two 
primary extensions to HAL. Specifically it extends the properties of HAL [*Link Objects*][] and adds a new reserved 
document keyword called "_meta" that defines document [*Reference Objects*][]. 

## 1. Introduction

There is an emergence of non-HTML HTTP applications ("Web APIs") which use hyperlinks to direct clients around their 
resources.

The JSON Hypertext Application Language Extension (*Hale*) is a standard which establishes conventions for expressing 
hypermedia documents with JSON ([RFC4627](https://www.ietf.org/rfc/rfc4627)).

*Hale* is a general purpose media type with which Web APIs can be developed. Clients of these APIs can interact with 
services by reference to their relation type and use the *Hale* document to progress through the application.  

*Hale's* conventions result in a uniform interface for serving and consuming hypermedia, enabling the creation of 
general-purpose libraries that can be re-used on any API utilizing *Hale*.

The primary design goals of *Hale* are completeness, generality, simplicity and machine-to-machine (M2M) friendliness, 
the later being a primary motivator for the extension of HAL. 
 
The base HAL media-type is purposefully light and requires referencing human-readable documentation or content 
negotiation for machine-readable external service descriptor documents (e.g. WADL, RAML, etc.) to understand link 
template values and their constraints, form-related attributes and their constraints, and to determine associated 
uniform interface methods of the protocol expected for the link.

Instead of forcing APIs to externalize service descriptors that lend themselves to tight-coupling or M2M clients having
to understand any arbitrary number of possible machine-readable, service descriptor formats, *Hale* inherently 
includes features so that a M2M client only needs to understand the *Hale* media-type to late-bind and interact with an 
API that implements these optional features. 

*Hale* can be applied to many different domains, and imposes the minimal amount of structure necessary to cover the key 
requirements of a hypermedia API.

## 2. Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and 
"OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://www.ietf.org/rfc/rfc2119).

## 3. *Hale* Documents
A *Hale* Document uses the format described in [RFC4627](https://www.ietf.org/rfc/rfc4627) and has the 
media type "application/vnd.hale+json".

The following extensions to HAL and new functionality are detailed in the following sections:

(1) Extends HAL [*Link Objects*][] to include the new properties "method", "data", "render", "enctype", and "target".

(2) Introduces *Hale* [*Data Objects*][] that specify metadata and constraints associated with values that can be set by
clients for URI template variables and request body attributes. 

(3) Extends HAL [*Resource Objects*][] with a new reserved property "_meta" that defines *Hale* [*Reference Objects*][].

(4) Introduces *Hale* [*Reference Objects*][] that include a single reserved property "_ref" for flexible re-usablility
 of metadata associated with a [*Resource Object*][] and extends [*Link Objects*][] and [*Data Objects*] to inherit
 from [*Reference Objects*][].

Basic Example:
```json
{
  "_meta": {
    "any": { "json": "object" }
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
    "agent": { "href": "/agent/1", "method": "GET", "render": "embed" },
    "customer": [ 
      { "href": "/customer/1", "method": "GET" }
    ]
  },
  "_embedded": {
    "customer": [
      {
        "_links": {
          "self": { "href": "/customer/1", "method": "GET" },
          "edit": { 
            "href": ".../{?user_id}",
            "method": "PUT",
            "enctype": "application/json",
            "render": "resource",
            "data": {
              "name": { "type": "string", "required": true },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true },
              "user_id": { "scope": "href", "required": true }
            }
          }
        },
        "name": "Tom",
        "send_info": "yes"
      }
    ]   
  }
}

```

would be interpreted by a client as:

```json
{
  "_meta": {
    "any": { "json": "object" }
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
    "agent": { "href": "/agent/1", "method": "GET", "render": "embed" },
    "customer": [ 
      { "href": "/customer/1", "method": "GET" }
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
            "render": "resource",
            "data": {
              "name": { "type": "string", "required": true, "value": "Tom" },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true, "value": "yes" },
              "user_id": { "scope": "href", "required": true }
            }
          }
        },
        "name": "Tom",
        "send_info": "yes"
      }
    ]   
  }
}
```

For an advanced example, see the example for [*Reference Objects*][].

## 4. Link Objects
*Hale* extends HAL *Link Objects* to inherit from *Hale* [*Reference Objects*][] and introduces the following new 
properties:

### 4.1. method
The "method" property is OPTIONAL.

It specifies the uniform-interface method(s) (e.g. HTTP.GET) to use to excercise the link. Valid values a string or 
array of strings.

### 4.2. data
The "data" property is OPTIONAL.

It is an object whose property names are those of associated URI template variables or a body properties and values
are *Hale* [*Data Objects*]. It specifies information about valid data for exercising the link.

Example:

```json
{
  "_links": {
    "search": {
      "href": ".../{?state}",
      "data": {
        "state": { "options": [ "AL", "...", "WY"], "in": true },
      }
    }
  }
}
```

### 4.3. render
The "render" property is OPTIONAL.

The following values are accepted:

* "embed" - Indicates the client SHOULD resolve the resource and append it to the "_embedded" object using the link
relation name as the property. 
    Servers MUST only use this directive for *safe, idempotent* links.

* "resource" - Indicates the client should populate a response body with all the values of the related "*Resource 
Object* prior to interacting with the related Link Object.  E.g., this directive allows a client to properly interact 
with an HTTP.PUT method.
    Servers SHOULD only use this directive for *unsafe* and *idempotent* requests.

### 4.4. enctype
The "enctype" property is OPTIONAL.

Specifies the media type that should be used to make the request and MUST be a list or a string.  If it is a string the 
server only accepts requests of that media-type. If it is a list, the server will accept any of the media-types 
specified.

Defaults to application/json.

### 4.5. target

The reserved "target" property is OPTIONAL.

"target" specifies that the information the subset of associated resource to return.  If the "type" property is 
JSON then "target" should specify an JSONPATH [http://goessner.net/articles/JsonPath/], if it is XML it 
should specify an XPATH [RFC 5261]. 
 
The "target" property allows [*Reference Objects*][] to be created from remote resources for use in 
generating "options" for input attributes.

## 5. Data Objects
*Hale* *Data Objects* inherit from *Hale* [*Reference Objects*][] and specify metadata and constraints associated with 
values that can be set by a client in exercising a link. 

Example:

```json
{
  "_links": {
    "self": { "href": "...", "method": "GET" },
    "search": {
      "href": ".../{?search_term,state}",
      "method": "GET",
      "data": {
        "state": { "options": ["AL", "...", "WY"], "multi": true }
      }
    },
    "create": {
      "href": ".../{?user}",
      "method": "POST",
      "enctype": "application/x-www-form-urlencoded",
      "data": {
        "user": { 
          "scope": "href", 
          "required": true 
        },
        "given_name": { 
          "minlength": 4, 
          "maxlength": 30, 
          "required": true,
          "profile": "http://alps.io/schema.org/Person#givenName" 
        },
        "family_name": { 
          "type": "string", 
          "profile": "http://alps.io/schema.org/Person#familyName" 
        },
        "parents": {
          "type": "array",
          "profile": "http://alps.io/schema.org/Person",
          "data": {
            "given_name": { 
              "minlength": 4, 
              "maxlength": 30, 
              "required": true,
              "profile": "http://alps.io/schema.org/Person#givenName" 
            },
            "family_name": { 
              "type": "string", 
              "profile": "http://alps.io/schema.org/Person#familyName" 
            },            
          }
        },
        "email_address": { 
          "type": "string:email", 
          "required": true
        },
        "phone": { 
          "type": "number:tel" 
        },
        "phone_ext": {
          "min": 0,
          "max": 6
        },
        "ssn": {
          "pattern": "^(\d{3}-?\d{2}-?\d{4}|XXX-XX-XXXX)$"
        },
        "home": {
          "type": "object",
          "required": false,
          "data": {
            "address": { "type": "string" },
            "city": { "type": "string" },
            "state": { "options": [ "AL", "...", "WY"], "in": true },
            "postal_code": { "type": "number" }
          }
        }
      }
    }
  }   
}
```

A *Data Object* has the following properties:

### 5.1. Data Properties

#### 5.1.1. type
The "type" property is OPTIONAL.

Specifies the expected type of the data. Servers MAY provide type information in the format:
`primitive_type{:data_type}`, where the `primitive_type` is a down-cased JSON primitive and the `data_type` is OPTIONAL. 
Clients should at least recognize `data_type` values associated with data-related  
[HTML 5 "input" element](http://www.w3.org/html/wg/drafts/html/master/forms.html#the-input-element) control types.

Default value is "string" if unspecified.

#### 5.1.2. data
The "data" property is OPTIONAL.

Specifying a "data" property supports recursively defining data properties of objects as data values.

#### 5.1.3. scope
The "scope" property is OPTIONAL.

Specifies the scope of the *Data Object*. Servers MAY specify "href" or "either".

If a [*Link Object*][] is templated and no *Data Object* with a property for the template variable name exists, a client 
should assume there are no constraints associated with the URI template variable.

Specifying "href" indicates the object only applies to URI template variables. 

Specifying the value "either" indicates the object applies to either a URI template variable of the same name and 
that a body property SHOULD be supplied according to the properties of the object.

If unspecified, the *Data Object* specifies information about property values to be submitted as part of the request 
body.

#### 5.1.4. profile
The "profile" property is OPTIONAL.

Its value is a string which is a URI that hints about the profile (as defined by [I-D.wilde-profile-link][]) of the 
related resource.

#### 5.1.5. value
The "value" property is OPTIONAL.

Specifies the current or default value.

### 5.2 Constraint Properties

#### 5.2.1. options
The "options" property is OPTIONAL.

If options is specified it MUST be a list. Options specifies possible for the associated data. The list can either be
an array of "strings" or an array of JSON objects keyed by the value to be used (this may need to be a Reference
Object) to specify more about what property to use).

#### 5.2.2. in
The "in" property is OPTIONAL.

In is a boolean. If "in" is specified the *Data Object* must specify "options".  The constraint "in" 
specifies that the value of a request parameter or attribute MUST be within the specified set of "options".

#### 5.2.3. min
The "min" property is OPTIONAL.

The constraint 'min' specifies that the value of a request parameter is below a certain value. If the value is a 
string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

#### 5.2.4. minlength
The "minlength" property is OPTIONAL.

Specifies the minimum length of a string, the minimum number of items in a list, or the minimum number of digits in a 
number.

#### 5.2.5. max
The "max" property is OPTIONAL.

The constraint 'max' specifies that the value of a request parameter is above a certain value. If the value is a 
string it specifies lexical order. If the value is a number it is treated as a numerical constraint.

#### 5.2.6. maxlength
The "maxlength" property is OPTIONAL.

Specifies the maximum length of a string, the maximum number of items in a list, or the maximum number of digits in a 
number.

#### 5.2.7. pattern
The "pattern" property is OPTIONAL.

Pattern specifies the PCRE regular expression that a string must conform to.

#### 5.2.8. multi
The "multi" property is OPTIONAL.

Multi is a boolean that specifies whether or not more than one of an item is allowed.  (i.e. ?name=foo&name=bar)

#### 5.2.9 required
The "required" property is OPTIONAL.

This object must be a boolean.  When a *Data Object" specifies the "required" as `true`, this attribute must be included 
in the request.

### 5.3 Constraint Extensions

Other attributes are considered constraint extensions. It may be an object or a reference. If it is a reference it must 
refer to something which conforms. Anything under a constrain extension MUST specify a profile tag giving a 
link that describes the constraint. The other attributes must match the tags specified in the profile.

## 6. Resource Objects
*Hale* adds an additional reserved property to HAL *Resource Objects*:

"_meta": contains reference metadata about the resource.

### 6.1. Reserved Properties

#### 6.1.1 _meta
The reserved "_meta" property is OPTIONAL.

It is an object whose properties provide information about the resource or resource attributes. The values must be a 
valid JSON object.

Defining a "_meta" attribute automatically defines a [*Reference Object*][] for the related *Resource Object*.

## 7. Reference Objects
*Hale* introduces *Reference Objects* that represent arbitrary JSON documents which MAY be referenced by other 
JSON objects inside a *Hale* document. All objects in a *Reference Object* are themselves *Reference Objects*. 

The primary purpose of *Reference Objects* is to simply, yet flexibly, provide a way to DRY documents and extend other 
*Hale* objects in meaningful and useful ways at runtime, effectively providing quasi-code-on-demand directives for
*Hale* aware clients to dynamically configure themselves.

It has one reserved property:

"_ref": contains an ordered array of directives about de-referencing related objects and resources.

Example:
```json
{
  "_meta": {
    "lookup": {
       "send_info": { "options": [ "yes", "no", "maybe" ], "in": true }
    },
    "edit_form": {
      "_ref": [ { "href": "/edit_form/1", "method": "GET", "type": "application/json" } ]
    }
  },
  "_links": {
    "self": { "href": "..." },
    "search": { 
      "href": ".../{?send_info}",
      "templated": true,
      "method": "GET",
      "data": { "_ref": [ "lookup" ] }
    },
    "agent": { "href": "/agent/1", "method": "GET", "render": "embed" },
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

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 273
Expires: Mon, 01 Jan 2525 00:00:00 GMT
ETag: "3e86-410-3596fbbc"

{
  "method": "PUT",
  "enctype": "application/json",
  "render": "resource",
  "data": {
    "name": { "type": "string", "required": true },
    "user_id": { "scope": "href", "required": true },
    "_ref": [ "lookup" ]
  }
}
```

```json
{
  "_meta": {
    "lookup": {
      "send_info": { "options": [ "yes", "no", "maybe" ], "in": true }
    },
    "edit_form": {
      "method": "PUT",
      "enctype": "application/json",
      "render": "resource",
      "data": {
        "name": { "type": "string", "required": true },
        "send_info": { "options": [ "yes", "no", "maybe" ], "in": true },
        "user_id": { "scope": "href", "required": true }
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
    "agent": { "href": "/agent/1", "method": "GET", "render": "embed" },
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
            "render": "resource",
            "data": {
              "name": { "type": "string", "required": true, "value": "Tom" },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true, "value": "yes" },
              "user_id": { "scope": "href", "required": true }
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
            "render": "resource",
            "data": {
              "name": { "type": "string", "required": true, "value": "Harry" },
              "send_info": { "options": [ "yes", "no", "maybe" ], "in": true, "value": "no" },
              "user_id": { "scope": "href", "required": true }
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

### 7.1. Reserved Properties

#### 7.1.1. _ref
The reserved "_ref" property is OPTIONAL.

Valid values are an array of:

(1) strings corresponding to properties of a particular *Reference Object* in a [*Resource Object*][] "_meta" tag.

(2) [*Link Objects*][] that should be de-referenced and it's attributes appended to the attributes
of the associated *Reference Objects*

A client SHOULD resolve all "_ref" declarations when encountered in a document according to the order of the array. 

A client MUST resolve a "\_ref" chain hierarchically starting at the top-level in the nearest "\_meta" tag of the 
current [*Resource Object*][]. Given the recursive structure of HAL, if the particular value is not discovered in the 
current [*Resource Object*][], the client SHOULD look in a [*Resource Object*][] embedding the current 
[*Resource Object*][], if any.

The inheritance hierarchy for common *Reference Objects* attributes is the value of the de-referencing 
*Reference Objects* supersedes those in the objects referenced.  

If a client cannot resolve the "_ref" attribute, it SHOULD treat it as a literal value.

##### 7.1.1.1. String values
A string value indicates the name of a Reference object property in a "_meta" tag in the current 
[*Resource Object*][] or another [*Resource Object*][] embedding the current resource 
object.

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

##### 7.1.1.2.  Link Object values
If a value is a *Hale* [*Link Object*][], the client SHOULD dereference the associated resource and content-negotiate 
media-types it understands, if a`type` attribute is not specified in the [*Link Object*][].

Unless a "target" property is specified in the [*Link Object*][], the client SHOULD completely fill in any 
attributes of the *Reference Object *with those of the return ed resource with the caveat that in the case of like 
attributes, the attribute of the [*Resource Object*][] supersedes that of the de-referenced resource. 

See [*Link Object*][] ["target"](#45-target) property for more information.

Example:

```json
{
  "_meta": {
    "monster" {
      "demeanor": "scary"
    },
    "explosion": {
      "occupation": "swamp thing",
      "_ref": [ { "href": "/human/1", "method": "GET", "type": "application/json" }, "monster" ]
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
  "occupation": "Scientist",
  "demeanor": "friendly"
}
```

```json
{
  "_meta": {
    "monster" {
      "demeanor": "scary"
    },
    "explosion": {
      "name": "Alex Olsen",
      "occupation": "Swamp Thing",
      "demeanor": "scary"
    }
  }
}
```

## 8. Authors

[Shea Valentine](https://github.com/sheavalentine-mdsol) and [Mark W. Foster](https://github.com/fosdev)

[*Data Objects*]: #7-data-objects
[*Data Object*]: #7-data-objects
[*Link Objects*]: #6-link-objects
[*Link Object*]: #6-link-objects
[*Reference Object*]: #5-reference-objects
[*Reference Objects*]: #5-reference-objects
[*Resource Object*]: #4-resource-objects
[*Resource Objects*]: #4-resource-objects
[I-D.wilde-profile-link]: http://tools.ietf.org/html/draft-wilde-profile-link-04
