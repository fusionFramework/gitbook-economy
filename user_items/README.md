# User items

This part of the guide discusses items that the user has, this means items that were loaded as a `Model_User_Item`.

You can retrieve user items in several ways:

Through the Item helper:
 - By location
 - By checking if the user has it


 Or by loading it directly:

~~~php
// Load the user item with id 1
$item = ORM::factory('User_Item', 1);

// Load the item by name and location from user with 1 as id
$item = ORM::Factory('User_Item')
	->where('item.name', '=', 'Apple')
	->where('user_item.location', '=', 'inventory')
	->where('user_id', '=', 1)
	->find();
~~~

> User info and Item info will always be loaded alongside your User item data

This means that if you need any information stored in the Item model you can access it through `$item->item`, the same
goes for user info `$item->user`.

Read [this article](https://github.com/jheathco/kohana-orm/wiki) if you're not too familiar with Kohana's ORM and relations.







## How to retrieve the item's name

There are several ways to access an item's name.

If you only need the name of the item as defined in the admin you should call

~~~php
	// would return apple
	$item->item->name
	~~~

If you want the item's name prefixed with its amount you should call

~~~php
	// would return 5 apples if the stack's amount is 5
	$item->name()
~~~

If you want to define the amount yourself you should call this

~~~php
	// would return 3 apples
	$item->item->name(3)

	// would return 1 apple
	$item->item->name(1)
~~~

## How to retrieve the item's image

If you need to access the item's image URL you can just call

~~~php
	$item->img();
	~~~

