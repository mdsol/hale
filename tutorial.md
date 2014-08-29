# Hale Tutorial
This document describes how the Hale serializer renders documents by providing examples using a ficitonal API, Coffee Bucks. The tutorial steps through every feature of Hale and provides concrete examples of how to use it.

## An Overview of Coffee Bucks
The Coffee Bucks APIrepresents the Point of Sale (PoS) system for a coffee shop.  

## The Basic Hale Document
The simplest valid Hale document is an empty object:
```json
{}
```
This is so because a hale document requires no elements.  Obviously this document contains no information, but it nonetheless provides a canvas from which we can work and that we will use in the following sections.

## The Coffee Bucks Entry Point
The first thing we want to do is to describe an API entry point. This entry point is the place in the API that allows client servers to perform the basic actions that exist on a PoS API. In the entry point we anticipate a set of links that go to the other actions.  So our basic structure looks like the following:
```json
{
    "_links": {}
}
```
Here we added the "_links" element.  The Hale spec defines the "_links" property as follows:
```
It is an object whose property names are link relation types (as defined by [RFC5988]). Its values are either a Link Object or an array of link objects.
```
While this may seem obtuse, there's nothing special happening here.  A link object is just like the links or forms you find on web pages.  That is, link objects describe relationships between resources.
As you might expect, the most basic link relation that every Hypermedia API should define is the "self" link.  So let's add that:

```json
{
    "_links": {
        "self": {
            "href": "www.example.com/coffeebucks/"
        }
    }
}
```
Now we have the first link in the Hale document.  In this case, we have defined the `self` link, which is a link that describes the resource you are currently on. Since we are on the entry point of the API, I have decided to make the URL `www.example.com/coffeebucks/`.  Let's go ahead and add the `profile` link as well:
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
The `profile` links to machine-readable documentation that describes the document we're looking at.  Since in this example we are rendering the documentation with the ALPS media-type, we put the `enctype` property under the `profile` link object to tell a client server to request that media-type when following that link.

But suppose we want to also provide an HTML page that was intended for human consumption in addition to the machine-readable ALPS document.  In that case, you provide the value of the `enctype` property that points to a list of enctypes.
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
This tells client servers that when they follow the `profile` link they can either request the profile as `application/alps+xml` or as `text/html` media-types.  By default, the client server is expected to request the current media-type; in this case, the media-type is `application/hale+json`.
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
The URL chosen here is the same as the `self` link because I've decided that another resource is superfluous. Still, this link will show all orders, regardless of their current state. What's probably more important for the business workflow is to show orders that are in a particular state. 

Let's go ahead and add that capability into our Hale document as follows:
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
With this addition to the Hale document, we have added the `order_status` template variable in the `href` property, as well as a matching `order_status` data object that has two properties. 
The `scope` property tells the client server whether the data belongs in the URL or in the request body. Since the `scope` value is `href`, the client knows that this `order_status` belongs in the URL.
The `options` property here specifies that valid values for `order_status` are `pending_payment`, `preparing`, or `fulfilled`. These are the "states" that an order can have. The `in` property specifies that those are the only possible fields.
We are also allowing multiple values to be set for `order_status`, which means that we are allowing URLs such as 'www.example.com/coffeebucks?order_status=preparing\&order_status=fulfilled' which would show a list of orders matching either of those.

#### Creating a Coffee Order
The other thing we're going to want to do is to create the coffee order.  Again, this is just another link object. However, since it's likely to be a more complicated link object we'll start by defining it separately from the context of the rest of the document. Then we can integrate that link object into the rest of the document at the end.
We'll define this order link object as follows:
 ```json
 {
     "place_order": {
         "href": "www.example.com/coffeebucks/orders",
         "method": "POST",
         "data": {}
     }
 }
 ```
This link object is the basic description of a form. We haven't yet added the data objects that specify the properties of a coffee order, but we are pointing again to `www.example.com/coffeebucks/orders`. This time, however, we are using the `POST` HTP method. By specifying this method we tell the client server that the API server expects a request body. The `request_encoding` isn't set here since it defaults to "application/x-www-form-urlencoded" but the property tells the API server that the request body from the client should be encoded as `application/x-www-form-urlencoded`.
For the purpose of this exercise, we'll define the following attributes as things that the client can specify to create a drink order: `drink_type`, `iced`, `size`, `shots`, and `decaf`.
Let's define each of these data objects one by one. For example, we define `drink_type` as follows:
 ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
Here we have specified the `drink_type` data object. Like `order_status` before, `drink_type` specifies a set of possible orders and also specifies it only accepts values within that set.  Unlike `order_status`, however, it is required, does not allow multiple values, and is in the request body instead of the URL. 
Let's add 'iced' next, as follows:
```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
The `iced` data object uses very different attributes. Here we specify `type`, which tells the client that the expected data type is a JSON Boolean type. The `value` property tells the client that if the `iced` property is not in the request body, the API server will assume that the value is `false`.  
Now let's add `size`, as follows:
```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
With the `size` data object we see `type` again show up. This time, though, it specifies a `primitive` type and a `data` type. Note that the the primitive type is specified before the colon (:) and that it specifies the JSON data type that it is supposed to be represented with. The data type is specified after the colon (':') and is intended to provide a _hint_ to the client about what sort of type this is. In this case, we specify that it is a range type, which is a well-defined HTML input type. It could be used by the client to understand that the elements in the `options` property have a rank ordering.

We also specify a profile.  This profile links to a specific element of the `coffeebucks` profile, which could specify how the numbers map to the names for those sizes.  Alternatively, you could render this as follows:
  ```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
This is a friendlier rendering.  Note that this rendering specifies instead that its primitive type is integer, and its HTML data type is number. 
  NOTE: This data type could be anything; for example, it could be data type `coffeesizes`. But Hale anticipates that the client always understands HTML types, whereas `coffeesizes` would require specialized client knowledge. 
For the options in the example, we defined a list of objects that give a name to each of the sizes and allows the client to "think" of the sizes either as numbers or as their name. 
Let's add the `shots` property next, as follows:
```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
The shots data object is a more sensible range input.  Here we define a minimum with `min` and we define a maximum with `max`.  This tells the client that the most shots ever allowed is 16 and the least ever allowed is 0.  Note that in this rendering we haven't specified `value`, nor have we specified `required`. This is because the API server will create a default number of shots based upon the size of the drink and the kind of drink that was requested. 
Finally, we add a similar object for `decaf`, as follows:
```json
{
    "place_order": {
        "href": "www.example.com/coffeebucks/orders",
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
The `decaf` data object looks almost exactly like the `shots` Data Object. But note that it uses a property called `lte`.  This property is not defined in the Hale spec; instead it is using section 5.3 Constraint Extensions. In this case, we want to tell the client that `decaf` should be less than `shots` since 'decaf' is intended to specify which proportion of those shots should be made with 'decaf' coffee.  However 'lte' is not in the spec, so if the client ignores this property it may happen to work - but should the client get an error from setting 'decaf' greater than 'shots', it can scrutinize the Data Objects, dereference the profile, and register the 'lte' Constraint Type.

####Putting It All Together
When we put the whole of the 'place_order' Link Object into the rest of our Hale document, we have the following:
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
So far, we've added the basic links for our API, but this Hale document isn't just here for providing links. We also want to provide the set of orders in the system. This means we need to add some data and some more controls.

#### Adding the orders data
It is likely that you are going to want to tell the client how many total orders there are, how many orders are currently being displayed. You're also going to want to paginate the results. 
In the following, we'll add those.
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
In this rendering, we add a `next` link that points to the current url with `?page=2` appended to the end. We also add a `page` attribute to the `orders` link, which enables the client to skip to a specific page. We also add the `count` and `total_count` attributes to the base document.
The client can understand these attributes by looking at the profile provided in the `profile` link.

#### Adding Order References
Finally, we need to put references to the orders in the document.
Here you might be tempted to define an `orders` attribute in the base of the document. Resist this temptation; instead put this information in the \"_links" section as follows:
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
In this rendering, we add an `order_list` element. We could call it `orders` and call the `orders` link `navigate` or something such as that. Remember that it doesn't really matter what you call them since their proper semantic meaning is described in the profile.
Unlike the other links, `order_list` is a JSON array rather than a JSON object. By specifying this array, you are saying that each link object within this array belongs to the `order_list` relation. 
 NOTE: This is most likely to come up when a resource acts as a container. But nothing prevents you from specifying two different ways to "search" using this structure.
#### Embedded Data
We now have a complete resource. However, given that our client is likely to be an interface, it might be worthwhile to have a way of including the order information in the resulting document. To do this, you first need to define an order resource as follows:
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
In this rendering, there is an `order` resource. It is a double, half-caf latte that is currently being prepared. There's also a link to change the state of the resource to `fulfilled`. This is a PUT relation, and no additional data about the 'status' attribute is provided, because the only valid value from this state is `fulfilled`.
It also defines "render" which instructs the client to take the current values of "www.example.com/coffeebucks/1" and prepopulate the data elements with them. If the server supported partial PUT's - treating PUT like PATCH then the "render" property would be unnecassary.
So, returning to our entry point, we can now embed these resources inside our base document as follows:
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
To summarize, to embed you create an array under `\_embedded` that is keyed by an attribute that matches the `\_link` attribute; in this case, `order_list`.  Each object in the 'order_list' has its own state. So it provides different links that are contingent on that state.

With this addition, you have successfully constructed a complex Hale document for a fully hypermedia, machine-driven, API to express a simple coffeebucks process. 

Now let's see what else we can do with a Hale document.

## Complex Objects
One of the first things you notice about the Coffee Bucks API is that it only allows clients to create one drink at a time. Clearly, though, people in the real world order many drinks. Fortunately there is a mechanism in Hale to support this real-world circumstance. That is, we can recursively define the object that is submitted in a link. To do this, we create another link relation that supports the submission of multiple drink orders.

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
This looks very much like the previously defined `place_order` attribute, but note several important changes.

First, notice that there's a different `request_encoding`. This is because the `form-urlencoded` attribute doesn't specify a way to POST complex data structures. Second, the `data` element contains two fields: `multi_order` and `orders`.  "The `mutli_order` field specifies to the server computer that it is going to have more than a single order. The `orders` field actually contains all the orders.

Looking at the `orders` field, note that its `type` is `object`. Being of an object type means that the `type` field is expecting a JSON object or its equivalent. It also specifies `multi` as true; this tells the client that the server anticipates in this case an array of objects.

The `orders` field specifies its own `data` field which then identifies the sub-elements exactly as before. So a client would send a request looking like the following, telling the coffee server to make a small regular and a small iced coffee:

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

## _meta 

With the preceding additions included, the Hale document now looks has the following information:

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

As is, the document will work. While the document provides all of our functionality, however, it is needlessly verbose. To tweak and tighten up the document, we can use Hale's `_meta` feature. The `_meta` feature allows you to specify arbitrary information _about_ the current resource, information that is not itself a property of that resource. For example, you can specify `store_location` as an arbitrary meta property of the resource. Store location is relevant to the resource, but it is not a propery of an order itself.

When a property is placed in `_meta` it automatically creates a _reference object_. Including a reference object in the definition of a state enables you to specify information that other resources in the document can use. In an endeavor to tighten up the document, you can use a `_meta` attribute consolidate the biggest chunk of the document, namely the common fields that the `place_order` and `multi_order` fields have.

The following changes reflect this use of the `_meta` feature to get this data.

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

In this emendation of the document, you'll note that we define a new reference object, `order_properties`, in the `_meta` section of the document. Once you define it, you can use `order_properties` throughout the document with the `_ref` tag. This `order_properties` tag always takes a list of tags which are interpreted from first to last item. These list tags instruct the client to replace that "_ref" tag with the listed properties.

## target
Now that the document is a little more terse, there is one other thing to consider. The `drink_type` element contains a menu. It may be out of the server's scope to know what that menu is and the menu may change. Therefore, instead of putting the menu inside the document explicitely, we can just reference an external resource to populate the document.

In the following rendering, we'll change `order_properties` to use a `_ref` object and get the list of menu items:

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

This change tells the client to populate options with the current menu items `item_names` and replace `options` under `drink_type` with the result.

Adding that, we'll also make the document a little more terse by using {"render": "embed"} instead of automatically rendering the items. We'll also add a `shot_base` in `_meta` to reduce that replication.

The following rendering shows what the Hale document now looks like:

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
With these changes, we have successfully rendered a Hale document that implements a Coffeebucks API.