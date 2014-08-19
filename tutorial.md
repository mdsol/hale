# Hale Tutorial
This document show how Hale renders documents by providing examples using the Coffee Bucks API.  It will go through every feature of Hale, providing a concrete example of how it's used.

## An Overview of Coffee Bucks
Coffee Bucks is an API that represents the Point of Sale system for a coffee shop.  

## The Basic Hale document
The most simple valid Hale document is an empty object
```json
{}
```
This is because none of the elements of a Hale document are required.  Obviously this document contains no information, but it nonetheless provides a canvas from which we will be working from.

## The Coffee Bucks entry point
The first thing we want to do, is describe an API entry point.  This is the place in the API which allows one to perform the basic actions that would exist on a Point of Sale.  Here we anticipate a set of links that go to the other actions.  So our basic structure is going to look like this:
```json
{
    "_links": {}
}
```
Here we have added the element \"_links".  The spec defines the \"_links" property as follows:
```
It is an object whose property names are link relation types (as
   defined by [RFC5988]) and values are either a Link Object or an array
   of Link Objects.
```
While this may seem obtuse, there's nothing special happening here.  A link object is just like the links or forms you find on web pages.  They describe relationships between resources.
The most basic link relation that very Hypermedia API should define is the "self" link.  So let's add that:

```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        }
    }
}
```
Now we have our first link in our Hale document.  In this case we have defines the 'self' link, which is a link describing the resource you are currently on. Since we are on the entry point of the API, I have decided to make the URL 'www.example.com/coffeebucks/'.  Let's go ahead and add the 'profile' link as well.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": "application/alps+xml"
        }
    }
}
```
The profile is this case links to machine readable documentation that describes the document we're looking at.  Since in this example we are rendering the documentation with the ALPS media type, we put the enctype property under the 'profile' Link Object to tell the Client to request that mediatype when following that link.
But suppose we wanted to also provide an HTML page that was intended for human consumption in addition to the machine readable ALPS.  In that case we have the value of the enctype property pointing to a list of enctypes.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        }
    }
}
```
This tells the client that when they are following the 'profile' link they can either request the profile as 'application/alps+xml' or as 'text/html'.  By default the client is expected to request the current mediatype in this case 'application/hale+json'

### Links that change state
There are two actions that one is able to take from the entry point.  They can either see a list of orders, or they can place a new order.

#### Adding a GET link
Let's start by specifying how to see a list of orders.  Using HTTP this is a GET request, which is the default 'method' that Hale assumes.  So adding this is as simple as adding the links we've seen before.

```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks"
        }
    }
}
```
The URL chosen here is the same as the self link, because I've decided that another resource would be superfluous, but this is going to show all orders regardless of their current state.  What's probably more interesting is to show orders that are in a particular state.  Let's go ahead and add that capability into our Hale document.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                }
            }
        }
    }
}
```
Here we have added the order_status template variable in the 'href' property, and the matching 'order_status' data object, which contains two properties.  The 'scope' property tells the client whether the data belongs in the URL or in the request body.  Since the 'scope' value is 'href', the client knows that this 'order_status' belongs in the URL.
The 'options' property here specifies that valid values for 'order_status' are 'pending_payment', 'preparing', or 'fulfilled' - which are the states an order can be in, and the 'in' property specifies that those are the only possible fields.
 We are also allowing multiple values to be set for 'order_status', which means we're allowing URL's such as 'www.example.com/coffeebucks?order_status=preparing\&order_status=fulfilled' which would show a list of orders matching either of those.
 
#### Creating a Coffee Order
 The other thing we're going to want to be able to do is actually create the coffee order.  Again this is just another Link Object, but since it's likely to be a more complicated Link Object we'll start by defining it separate from the context of the rest of the document, and then integrate it into the rest of the document at the end.
 ```json
 {
     "place_order": {
         "href": "www.example.com/coffeebucks/orders",
         "request_encoding": "application/x-www-form-urlencoded",
         "method": "POST",
         "data": {}
     }
 }
 ```
 This is the basic description of a form. We haven't yet added the Data Objects that specify the properties of a Coffee Order, but we are pointing again to 'www.example.com/coffeebucks/orders', however this time we are using the 'POST' method. By specifying this method we are telling the client that the server is expecting a request body; the request_encoding tells the server that the request body should be encoded as 'application/x-www-form-urlencoded'.
 For the purpose of this exercise we'll define the following attributes as things which the client can specify to create a drink order: drink_type, iced, size, shots, decaf.  Let's define each of these Data Objects one by one.
 ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            }
        }
    }
}
  ```
  Here we have specified the 'drink_type' Data Object.  Like 'order_status' before it specifies a set of possible orders and that it only accepts values within that set.  Unlike 'order_status' it is required, does not allow multiple values, and is intended to be in the request body instead of the URL.  Let's add 'iced' next.
 ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            }
        }
    }
}
  ```
  The 'iced' Data Object uses very different attributes. Here 'type' is specified, telling the client that the expected datatype is a JSON Boolean type. The 'value' property tells the client that if the 'iced' property is not sent in the request body, it will assume that the value is false.  Now let's add size.
multiple values, and is intended to be in the request body instead of the URL.  Let's add 'iced' next.
  ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:range",
                "profile": "profiles.example.com/coffeebucks#sizes",
                "options": [
                    8,
                    12,
                    16,
                    20
                ],
                "in": true,
                "required": true
            }
        }
    }
}
  ```
  With the 'size' Data Object we see type again show up.  This time it specifies both a 'primitive' type, but also a 'data' type.  The primitive type is specified before the colon (':'), and specifies the JSON Datatype that it is supposed to be represented with. The data type is specified after the colon (':') and is intended to provide a _hint_ to the client about what sort of type this is. In this case we've specified that it is a Range type, which is a well defined HTML Input type, and could be used by the client to understand that the elements in the 'options' property have rank ordering.
  We also specify a profile here.  This links to a specific element of the coffeebucks profile, which could specify how the numbers map to the names for those sizes.  Alternatively it could have been rendered as follows.
  ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:number",
                "options": [
                    {
                        "small": 8
                    },
                    {
                        "medium": 12
                    },
                    {
                        "large": 16
                    },
                    {
                        "extra-large": 20
                    }
                ],
                "in": true,
                "required": true
            }
        }
    }
}
  ```
  This is a more friendly rendering.  Here it specifies instead that its primitive type is Integer, and its HTML datatype is number (this could be anything, it could be datatype 'coffeesizes', but Hale anticipates that the client always understands HTML types, whereas coffeesizes requires specialized client knowledge).  For the options we have defined a list of objects that give a name to each of the sized, and allows the Client to "think" of the sizes either as numbers or as their name. Let's look at the 'shots' property next.
  ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:number",
                "options": [
                    {
                        "small": 8
                    },
                    {
                        "medium": 12
                    },
                    {
                        "large": 16
                    },
                    {
                        "extra-large": 20
                    }
                ],
                "in": true,
                "required": true
            },
            "shots": {
                "type": "integer:range",
                "min": 0,
                "max": 16
            }
        }
    }
}
  ```  
  The shots Data Object is a more sensible Range input.  Here we define a minimum with 'min', and we define a maximum with 'max'.  This tells the client that the most shots ever allowed in 16, and the least ever allowed is 0.  Here we haven't specified 'value', nor have we specified 'required' because the server will create a default number of shots based upon the size of the drink and the kind of drink requested.  We're going to use a similar Object for 'decaf'.
  ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/x-www-form-urlencoded",
        "method": "POST",
        "data": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:number",
                "options": [
                    {
                        "small": 8
                    },
                    {
                        "medium": 12
                    },
                    {
                        "large": 16
                    },
                    {
                        "extra-large": 20
                    }
                ],
                "in": true,
                "required": true
            },
            "shots": {
                "type": "integer:range",
                "min": 0,
                "max": 16
            },
            "decaf": {
                "type": "integer:range",
                "min": 0,
                "max": 16,
                "lte": {
                    "profile": "profiles.example.com/lte/",
                    "attribute": "shots"
                }
            }
        }
    }
}
  ```  
  The 'decaf' Data Object looks almost exactly like the 'shots' Data Object, except that it is using a property called 
  'lte'.  This property is not defined in the Hale spec, instead it is using section 5.3 Constraint Extensions. In this case we want to tell the client that 'decaf' should be less than 'shots', since 'decaf' is intended to specify which proportion of those shots should be made with 'decaf' coffee.  However 'lte' is not in the spec, so if the client ignores this property it may happen to work - but should the client get an error from setting 'decaf' greater than 'shots', it can scrutinize the Data Objects, dereference the profile, and register the 'lte' Constraint Type.
  When we put the whole of the 'place_order' Link Object into the rest of our document, we end up with this.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                }
            }
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "drink_type": {
                    "options": [
                        "coffee",
                        "americano",
                        "latte",
                        "mocha",
                        "cappuccino",
                        "macchiato",
                        "espresso"
                    ],
                    "in": true,
                    "required": true
                },
                "iced": {
                    "type": "boolean",
                    "value": false
                },
                "size": {
                    "type": "integer:number",
                    "options": [
                        {
                            "small": 8
                        },
                        {
                            "medium": 12
                        },
                        {
                            "large": 16
                        },
                        {
                            "extra-large": 20
                        }
                    ],
                    "in": true,
                    "required": true
                },
                "shots": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16
                },
                "decaf": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16,
                    "lte": {
                        "profile": "profiles.example.com/lte/",
                        "attribute": "shots"
                    }
                }
            }
        }
    }
}
```  
This gives us the basic links for our API, but this page isn't just here for providing links, we also want to provide the set of orders in the system.  This means we need to add some data, and some more controls.

#### Adding the orders data
It is likely that you are going to want to tell the client how many orders there are total, how many orders are currently being displayed, and you're going to want pagination.  So let's add that.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "drink_type": {
                    "options": [
                        "coffee",
                        "americano",
                        "latte",
                        "mocha",
                        "cappuccino",
                        "macchiato",
                        "espresso"
                    ],
                    "in": true,
                    "required": true
                },
                "iced": {
                    "type": "boolean",
                    "value": false
                },
                "size": {
                    "type": "integer:number",
                    "options": [
                        {
                            "small": 8
                        },
                        {
                            "medium": 12
                        },
                        {
                            "large": 16
                        },
                        {
                            "extra-large": 20
                        }
                    ],
                    "in": true,
                    "required": true
                },
                "shots": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16
                },
                "decaf": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16,
                    "lte": {
                        "profile": "profiles.example.com/lte/",
                        "attribute": "shots"
                    }
                }
            }
        }
    },
    "count": 3,
    "total_count": 6
}
``` 
Here we've added a 'next' link that points to the current url with "?page=2" appended to the end.  We've also added a 'page' attribute to the 'orders' link, allowing the client to skip to a specific page.
Also, we've added the 'count' and 'total_count' attributes to the base document.  The client is capable of understanding these attributes by looking at the profile provided by the 'profile' link.
Finally, we need to actually put the references to the orders in the document.
Here it might be tempting to define a 'orders' attribute in the base of the document.  Resist this temptation; instead we want to put this information in our \"_links" section.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "scope": "href",
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "drink_type": {
                    "options": [
                        "coffee",
                        "americano",
                        "latte",
                        "mocha",
                        "cappuccino",
                        "macchiato",
                        "espresso"
                    ],
                    "in": true,
                    "required": true
                },
                "iced": {
                    "type": "boolean",
                    "value": false
                },
                "size": {
                    "type": "integer: number",
                    "options": [
                        {
                            "small": 8
                        },
                        {
                            "medium": 12
                        },
                        {
                            "large": 16
                        },
                        {
                            "extra-large": 20
                        }
                    ],
                    "in": true,
                    "required": true
                },
                "shots": {
                    "type": "integer: range",
                    "min": 0,
                    "max": 16
                },
                "decaf": {
                    "type": "integer: range",
                    "min": 0,
                    "max": 16,
                    "lte": {
                        "profile": "profiles.example.com/lte/",
                        "attribute": "shots"
                    }
                }
            }
        },
        "order_list": [
            {
                "href": "www.example.com/coffeebucks/1"
            },
            {
                "href": "www.example.com/coffeebucks/2"
            },
            {
                "href": "www.example.com/coffeebucks/3"
            }
        ]
    },
    "count": 3,
    "total_count": 6
}
```
Here we've added an 'order_list' element.  We could have called in 'orders', and made our 'orders' link instead be called 'navigate' or something, but it doesn't really matter what we call them - their proper semantic meaning is described in the profile.
Unlike the other links, 'order_list' is a JSON array rather than a JSON object.  When specifying this, you are saying each Link Object within this array belongs to the 'order_list' relation.  This is most likely to come up when a resource is acting as a container, but there is nothing to prevent you from specifying two different ways to 'search' using this structure.
#### Embedded Data
We now have a complete resource.  However, given that our client is likely to be an interface, it might be worthwhile to have a way of including the order information in the document.  To do this first we'll need to define an order object.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/1"
        },
        "fulfill": {
            "href": "www.example.com/coffeebucks/1",
            "method": "PUT",
            "render": "prepopulate",
            "data": {
                "status": {
                    "value": "fulfilled",
                    "required": true
                }
            }
        }
    },
    "status": "preparing",
    "created": 12569537329,
    "drink_type": "latte",
    "iced": "true",
    "size": 8,
    "shots": 2,
    "decaf": 1
}
```
Here we have an order resource.  It is a double half-caf latte that is currently being prepared, and provides a link to change the state of the resource to 'fulfilled'.  This is a PUT relation, and no additional data about the 'status' attribute is provided, because the only valid value from this state is 'fulfilled'.
It also defines "render" which instructs the client to take the current values of "www.example.com/coffeebucks/1" and prepopulate the data elements with them. If the server supported partial PUT's - treating PUT like PATCH then the "render" property would be unnecassary.
So, returning to our entry point, we can now embed these resources inside our base document as follows.
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "drink_type": {
                    "options": [
                        "coffee",
                        "americano",
                        "latte",
                        "mocha",
                        "cappuccino",
                        "macchiato",
                        "espresso"
                    ],
                    "in": true,
                    "required": true
                },
                "iced": {
                    "type": "boolean",
                    "value": false
                },
                "size": {
                    "type": "integer:number",
                    "options": [
                        {
                            "small": 8
                        },
                        {
                            "medium": 12
                        },
                        {
                            "large": 16
                        },
                        {
                            "extra-large": 20
                        }
                    ],
                    "in": true,
                    "required": true
                },
                "shots": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16
                },
                "decaf": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16,
                    "lte": {
                        "profile": "profiles.example.com/lte/",
                        "attribute": "shots"
                    }
                }
            }
        },
        "order_list": [
            {
                "href": "www.example.com/coffeebucks/1"
            },
            {
                "href": "www.example.com/coffeebucks/2"
            },
            {
                "href": "www.example.com/coffeebucks/3"
            }
        ]
    },
    "_embedded": {
        "order_list": [
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/1"
                    },
                    "fulfill": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "fulfilled",
                                "required": true
                            }
                        }
                    }
                },
                "status": "preparing",
                "created": 12569537329,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/2"
                    }
                },
                "status": "fulfilled",
                "created": 12569537312,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/3"
                    },
                    "prepare": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "preparing",
                                "required": true
                            },
                            "paid": {
                                "value": 495,
                                "required": true
                            }
                        }
                    },
                    "cancel": {
                        "href": "www.example.com/coffeebucks/3",
                        "method": "DELETE"
                    }
                },
                "status": "pending_payment",
                "created": 12569534536,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1,
                "cost": 495
            }
        ]
    },
    "count": 3,
    "total_count": 6
}
```
To embed, we have created an array under '\_embedded' that is keyed by an attribute that matches the '\_link' attribute - in this case 'order_list'.  Each object in the 'order_list' has it's own state, and so provides different links contingent on that state.
With this, we have successfully constructed a complex Hale document for a fully Hypermedia - machine driven - API to express a simple coffeebucks process.  Now let's see what else we can do with this.

## complex objects
One of the first things one can notice about the above API is that it only allows a single drink to be created at a time.  However, it's clear that people in the real world order many drinks; fortunately there is a mechanism for hale to support this.  This is done my recursively defining the object to be submitted in a link.  Here we'll create another link relation that supports multiple drink orders.

```json
{
    "multi_order": {
        "href": "www.example.com/coffeebucks/orders",
        "request_encoding": "application/json",
        "method": "POST",
        "data": {
            "multi_order": {
                "value": true
            },
            "orders": {
                "type": "object",
                "multi": "true",
                "data": {
                    "drink_type": {
                        "options": [
                            "coffee",
                            "americano",
                            "latte",
                            "mocha",
                            "cappuccino",
                            "macchiato",
                            "espresso"
                        ],
                        "in": true,
                        "required": true
                    },
                    "iced": {
                        "type": "boolean",
                        "value": false
                    },
                    "size": {
                        "type": "integer: number",
                        "options": [
                            {
                                "small": 8
                            },
                            {
                                "medium": 12
                            },
                            {
                                "large": 16
                            },
                            {
                                "extra-large": 20
                            }
                        ],
                        "in": true,
                        "required": true
                    },
                    "shots": {
                        "type": "integer: range",
                        "min": 0,
                        "max": 16
                    },
                    "decaf": {
                        "type": "integer: range",
                        "min": 0,
                        "max": 16,
                        "lte": {
                            "profile": "profiles.example.com/lte/",
                            "attribute": "shots"
                        }
                    }
                }
            }
        }
    }
}
```
This looks very much like previously defined "place_order", but here there are several important changes.
The first thing you'll notice is that it has a different request_encoding, because "form-urlencoded" doesn't specify a way to POST complex data structures.  The data element contains two fields: multi_order and order.  "mutli_order" is there to specify to the server that it is going to have more than a single order, "orders" will actually contain all the orders.
Looking at "orders", note that the "type" is an object.  Type "object" means that it's expecting a JSON Object (or equivalent).  It also specifies "multi" as true, which tells the client that it anticipates in this case an array of objects.
"orders" just like the request specifies its own "data" field which then specifies the sub-elements exactly as before.  So a client would send a request looking like this:
```json
{
    "multi_order": true,
    "orders": [
        {
            "drink_type": "coffee",
            "size": 8
        },
        {
            "drink_type": "coffee",
            "iced": true,
            "size": 8
        }
    ]
}
```
Which of course is telling the server to make a small regular and a small iced coffee.

## _meta 
Our whole document now looks like this
```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "drink_type": {
                    "options": [
                        "coffee",
                        "americano",
                        "latte",
                        "mocha",
                        "cappuccino",
                        "macchiato",
                        "espresso"
                    ],
                    "in": true,
                    "required": true
                },
                "iced": {
                    "type": "boolean",
                    "value": false
                },
                "size": {
                    "type": "integer:number",
                    "options": [
                        {
                            "small": 8
                        },
                        {
                            "medium": 12
                        },
                        {
                            "large": 16
                        },
                        {
                            "extra-large": 20
                        }
                    ],
                    "in": true,
                    "required": true
                },
                "shots": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16
                },
                "decaf": {
                    "type": "integer:range",
                    "min": 0,
                    "max": 16,
                    "lte": {
                        "profile": "profiles.example.com/lte/",
                        "attribute": "shots"
                    }
                }
            },
            "multi_order": {
                "href": "www.example.com/coffeebucks/orders",
                "request_encoding": "application/json",
                "method": "POST",
                "data": {
                    "multi_order": {
                        "value": true
                    },
                    "orders": {
                        "type": "object",
                        "multi": "true",
                        "data": {
                            "drink_type": {
                                "options": [
                                    "coffee",
                                    "americano",
                                    "latte",
                                    "mocha",
                                    "cappuccino",
                                    "macchiato",
                                    "espresso"
                                ],
                                "in": true,
                                "required": true
                            },
                            "iced": {
                                "type": "boolean",
                                "value": false
                            },
                            "size": {
                                "type": "integer:number",
                                "options": [
                                    {
                                        "small": 8
                                    },
                                    {
                                        "medium": 12
                                    },
                                    {
                                        "large": 16
                                    },
                                    {
                                        "extra-large": 20
                                    }
                                ],
                                "in": true,
                                "required": true
                            },
                            "shots": {
                                "type": "integer:range",
                                "min": 0,
                                "max": 16
                            },
                            "decaf": {
                                "type": "integer:range",
                                "min": 0,
                                "max": 16,
                                "lte": {
                                    "profile": "profiles.example.com/lte/",
                                    "attribute": "shots"
                                }
                            }
                        }
                    }
                }
            }
        },
        "order_list": [
            {
                "href": "www.example.com/coffeebucks/1"
            },
            {
                "href": "www.example.com/coffeebucks/2"
            },
            {
                "href": "www.example.com/coffeebucks/3"
            }
        ]
    },
    "_embedded": {
        "order_list": [
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/1"
                    },
                    "fulfill": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "fulfilled",
                                "required": true
                            }
                        }
                    }
                },
                "status": "preparing",
                "created": 12569537329,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/2"
                    }
                },
                "status": "fulfilled",
                "created": 12569537312,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/3"
                    },
                    "prepare": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "preparing",
                                "required": true
                            },
                            "paid": {
                                "value": 495,
                                "required": true
                            }
                        }
                    },
                    "cancel": {
                        "href": "www.example.com/coffeebucks/3",
                        "method": "DELETE"
                    }
                },
                "status": "pending_payment",
                "created": 12569534536,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1,
                "cost": 495
            }
        ]
    },
    "count": 3,
    "total_count": 6
}
```

This document, while providing all of our functionality, is needlessly verbose.  To remedy this we are going to lean on Hale's "_meta" feature.  "_meta" provides a way of specifying arbitrary information _about_ the current resource which is not itself a property of that resource.  For example, we might specify "store_location" as an arbitrary meta property of the resource - store location is relevant to the resource, but is not itself a propery of an order.
Whenever a property is placed in "_meta" it automatically creates a "Reference Object", which is a way of specifying information that can be used by other resources within the document.  In our endeavor to clean up the document, we will start by getting the biggest target - namely the common fields in "multi_order" and "place_order".

```json
{
    "_meta": {
        "order_properties": {
            "drink_type": {
                "options": [
                    "coffee",
                    "americano",
                    "latte",
                    "mocha",
                    "cappuccino",
                    "macchiato",
                    "espresso"
                ],
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:number",
                "options": [
                    {
                        "small": 8
                    },
                    {
                        "medium": 12
                    },
                    {
                        "large": 16
                    },
                    {
                        "extra-large": 20
                    }
                ],
                "in": true,
                "required": true
            },
            "shots": {
                "type": "integer:range",
                "min": 0,
                "max": 16
            },
            "decaf": {
                "type": "integer:range",
                "min": 0,
                "max": 16,
                "lte": {
                    "profile": "profiles.example.com/lte/",
                    "attribute": "shots"
                }
            }
        }
    },
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "_ref": [
                    "order_properties"
                ]
            },
            "multi_order": {
                "href": "www.example.com/coffeebucks/orders",
                "request_encoding": "application/json",
                "method": "POST",
                "data": {
                    "multi_order": {
                        "value": true
                    },
                    "orders": {
                        "type": "object",
                        "multi": "true",
                        "data": {
                            "_ref": [
                                "order_properties"
                            ]
                        }
                    }
                }
            }
        },
        "order_list": [
            {
                "href": "www.example.com/coffeebucks/1"
            },
            {
                "href": "www.example.com/coffeebucks/2"
            },
            {
                "href": "www.example.com/coffeebucks/3"
            }
        ]
    },
    "_embedded": {
        "order_list": [
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/1"
                    },
                    "fulfill": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "fulfilled",
                                "required": true
                            }
                        }
                    }
                },
                "status": "preparing",
                "created": 12569537329,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/2"
                    }
                },
                "status": "fulfilled",
                "created": 12569537312,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1
            },
            {
                "_links": {
                    "self": {
                        "href": "www.example.com/coffeebucks/3"
                    },
                    "prepare": {
                        "href": "www.example.com/coffeebucks/1",
                        "method": "PUT",
                        "data": {
                            "status": {
                                "value": "preparing",
                                "required": true
                            },
                            "paid": {
                                "value": 495,
                                "required": true
                            }
                        }
                    },
                    "cancel": {
                        "href": "www.example.com/coffeebucks/3",
                        "method": "DELETE"
                    }
                },
                "status": "pending_payment",
                "created": 12569534536,
                "drink_type": "latte",
                "iced": "true",
                "size": 8,
                "shots": 2,
                "decaf": 1,
                "cost": 495
            }
        ]
    },
    "count": 3,
    "total_count": 6
}
```

Here you'll note that we define "order_properties" under the "_meta" section of the document.  This defines an "order_properties" reference object, which can be used anywhere in the document with the "_ref" tag.  The "_ref" tag always takes a list of tags (which are interpreted in from first to last), and these instruct the client to replace that "_ref" tag with the listed properties.

## target
Now that the document is a little more terse, there is one other thing to consider.  The "drink_type" contains a menu.  It may be out of the servers scope to know what that menu is, and the menu may change; so instead of putting the menu inside the document explicitely, we can instead reference an external resource to populate the document.
Here we'll change "order_properties" to use a "_ref" object and get the list of menu items
```json
{
    "order_properties": {
        "drink_type": {
            "options": {
                "_ref": [
                    {
                        "href": "www.example.com/hq/menu",
                        "type": "application/xml",
                        "target": "/menu/menuitem[current=true]/item_name[text()]"
                    }
                ]
            },
            "in": true,
            "required": true
        }
    }
}
```
This tells the client to populate options with the current menu items "item_name"s, and replace "options" under "drink_type" with the result.

Adding that we'll make the document a little more terse by using {"render": "embed"} instead of automatically rendering the items, and we'll add a "shot_base" in "_meta" to reduce that replication.
The resulting document looks like this:

```json
{
    "_meta": {
        "shot_base": {
            "type": "integer:range",
            "min": 0,
            "max": 16
        },
        "order_properties": {
            "drink_type": {
                "options": {
                    "_ref": [
                        {
                            "href": "www.example.com/hq/menu",
                            "type": "application/xml",
                            "target": "/menu/menuitem[current=true]/item_name[text()]"
                        }
                    ]
                },
                "in": true,
                "required": true
            },
            "iced": {
                "type": "boolean",
                "value": false
            },
            "size": {
                "type": "integer:number",
                "options": [
                    {
                        "small": 8
                    },
                    {
                        "medium": 12
                    },
                    {
                        "large": 16
                    },
                    {
                        "extra-large": 20
                    }
                ],
                "in": true,
                "required": true
            },
            "shots": {
                "_ref": [
                    "shot_base"
                ]
            },
            "decaf": {
                "_ref": [
                    "shot_base"
                ],
                "lte": {
                    "profile": "profiles.example.com/lte/",
                    "attribute": "shots"
                }
            }
        }
    },
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        },
        "profile": {
            "href": "profiles.example.com/coffeebucks/",
            "enctype": [
                "application/alps+xml",
                "text/html"
            ]
        },
        "orders": {
            "href": "www.example.com/coffeebucks{?order_status,page}",
            "data": {
                "order_status": {
                    "scope": "href",
                    "options": [
                        "pending_payment",
                        "preparing",
                        "fulfilled"
                    ],
                    "in": true,
                    "multi": true
                },
                "page": {
                    "type": "integer",
                    "min": 1,
                    "max": 2,
                    "value": 1
                }
            }
        },
        "next": {
            "href": "www.example.com/coffeebucks?page=2"
        },
        "place_order": {
            "href": "www.example.com/coffeebucks/orders",
            "request_encoding": "application/x-www-form-urlencoded",
            "method": "POST",
            "data": {
                "_ref": [
                    "order_properties"
                ]
            },
            "multi_order": {
                "href": "www.example.com/coffeebucks/orders",
                "request_encoding": "application/json",
                "method": "POST",
                "data": {
                    "multi_order": {
                        "value": true
                    },
                    "orders": {
                        "type": "object",
                        "multi": "true",
                        "data": {
                            "_ref": [
                                "order_properties"
                            ]
                        }
                    }
                }
            }
        },
        "order_list": [
            {
                "href": "www.example.com/coffeebucks/1",
                "render": "embed"
            },
            {
                "href": "www.example.com/coffeebucks/2",
                "render": "embed"
            },
            {
                "href": "www.example.com/coffeebucks/3",
                "render": "embed"
            }
        ]
    },
    "count": 3,
    "total_count": 6
}
```
And thus we have successfully rendered a Hale document implementing a Coffeebucks API.