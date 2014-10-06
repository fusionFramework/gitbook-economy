# Items

Items are an essential part of your game's economy. As such the item engine provided will give you a flexible starting point.

## Item model

Using this model you can load items that you've defined in your admin:

~~~php
    // Load item by id
    $item = ORM::factory('Item', 1);

    // Load item by name
    $item = ORM::factory('Item')
        ->where('item.name', '=', 'Apple')
        ->find();
~~~
### Getting the name

There are 2 ways of getting the name of the item, the most simple on being:

~~~php

    // would output Apple
    $item->name
    ~~~

A helper was added to the model in case you want to prefix the item's name with a number:

~~~php
    // outputs 2 apples
    $item->name(2);

    // outputs 1 apple
    $item->name(1);
~~~

### Getting the image
Item's images are always stored in the item type's directory, therefor a handy helper was added to retrieve the correct url
for every item's image:

~~~php
    $item->img();
~~~

### Check if the item is in circulation

An item can have one of several statuses:

 - Draft (when you're creating the item)
 - Retired (when you don't want to distribute the item anymore)
 - Released

If you want to check if the item is actually released you can call this method:
~~~php
    if ($item->in_circulation())
    {
        // Item is in circulation
    }
    else
    {
        // BOO
    }
~~~
