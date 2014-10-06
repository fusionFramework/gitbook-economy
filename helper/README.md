# Item helper

The item helper aims to make it easier for the developer to look for and give out items.

Most of the time you start of by calling the factory method which takes 1 argument.
This argument is either the item's id (in this case `Model_Item` will look for an existing item) or a loaded Model_Item instance:

~~~php
// Load the item with id 1
$helper = Item::factory(1);

// Load the item through a Model_Item instance
$item = ORM::Factory('Item')
	->where('name', '=', 'Apple')
	->find();
$helper = Item::factory($item);
~~~

> This will throw an `Item_Exception_Load` if an error occurs.

Now that you've provided the item helper with an item we can start having some fun.

As a little note, you can always access the loaded Model_Item from the helper like this:

		$item = $helper->item();

## Giving the item to a user

At some point in your game you might want to give the user items after doing something (playing a mini-game, quests,...),
this method is designed for that. It also automatically logs this action if it's done successfully (the log gets returned on success).

### Parameters
|Parameter      |Description                                                                                    |
|---------------|-----------------------------------------------------------------------------------------------|
|$user          |null if you want to send it to the logged in user, and id or ModelUser instance otherwise      |
|$origin        |Where is this transaction happening (for logs)                                                 |
|$amount        |How many copies does the user get (defaults to 1)                                              |
|$location      |Where are we sending the items to? (defaults to inventory)                                     |
|$ignore_limit  |Ignore the space limit of that location? (defaults to false), can be handy when sending rewards|

### Exceptions
|Class thrown           |Description                                                                                    |
|-----------------------|-----------------------------------------------------------------------|
|Item_Exception_Amount  | When an invalid or negative number is specific for $amount parameter  |
|Item_Exception_User    | When no user can be loaded                                            |
|Item_Exception_Space   | When the user hit the space limit for $location                       |
|Item_Exception         | A catch-all exception if you just want to catch 1 exception)          |

### Examples

Now, let's say the logged in user wins an 3 copies of an item in daily. If the user's inventory is full we'd still want to give him the winnings:

~~~php
$log = $helper->to_user(null, 'daily', 3, 'inventory', true);
~~~

Now let's say the user buys an item from a store, if he does not have any space left in his inventory we won't go through with the sale:

~~~php
try {
	$log = $helper->to_user(null, 'store', 1);

	// Seems to go through successful
}
catch(Item_Exception_Space $e)
{
	//seems like there's no space in the inventory to place the item, return an error
}
~~~

Let's do the same but account for all the exceptions that this method could throw and set them as `RD` warnings:

~~~php
try {
	$log = $helper->to_user(null, 'store', 1);
	RD::success('Thanks for buying :$item_name', [':item_name' => $helper->item()->name]);
}
catch(Item_Exception_Amount $e)
{
	RD::warning('The provided amount is incorrect');
}
catch(Item_Exception_User $e)
{
	// In this case it would only be fired if no user is logged in
	RD::warning('You need to be logged in to buy an item');
}
catch(Item_Exception_Space $e)
{
	RD::warning('You have no more space left in your inventory');
}
~~~

## Check if the user has the item

Sometimes it's necesary to check if the user an item (for example during a quest).
If this method is run successfully it will return the user item, otherwise it will return false.

### Parameters

|Parameter      |Description                                                                              |
|---------------|-----------------------------------------------------------------------------------------|
|$amount        |How many copies does the user need to have of this item? (defaults to 1)                 |                             |
|$location      |Where are should the item be located? (defaults to inventory)                            |
|$user          |null if you want to check the logged in user, otherwise an id or ModelUser instance      |

### Exceptions
|Class thrown           |Description                                                                                    |
|-----------------------|--------------------------------------------------------------|
|Item_Exception_Amount  | When an invalid or negative number is specific for $amount   |
|Item_Exception_User    | When no user can be loaded                                   |
|Item_Exception         | A catch-all exception if you just want to catch 1 exception) |

### Examples

So let's get down to buisines and check if the logged in user has at least 1 copy ofthe item in his inventory:

~~~php
$item = $helper->user_has();

if($item == FALSE)
{
	// Nope, user does not have it
}
else
{
	// Yup the user has it, let's echo the amount of copies he has.
	echo $item->amount;
}
~~~

Now let's check if he has 3 copies in his safe, this time we'll also check for exceptions:

~~~php
try {
	$item = $helper->user_has(3, 'safe');

	if($item == FALSE)
	{
		// Nope, user does not have it
	}
	else
	{
		// Yup the user has it, let's echo the amount of copies he has.
		echo $item->amount;
	}
}
catch(Item_Exception_Amount $e)
{
	RD::warning('The provided amount is incorrect');
}
catch(Item_Exception_User $e)
{
	// In this case it would only be fired if no user is logged in
	RD::error('No one seems to be logged in!');
}
~~~

## How check if the user has several items at once

This static helper method tries to retrieve a set of items based on the provide item's id.

It does not guarantee that he user has all of them, it will return an array for you to loop over and check yourself,
it just simplifies finding them.

It will return an array, if an item was found it will be added to the array
(the key will be the same as the item's id, the value will be a loaded `Model_User_Item` instance).

### Parameters

|Parameter    |Description                                                    |
|-------------|---------------------------------------------------------------|
|$item_ids    |An array with item's id                                        |
|$location    |Where are should the item be located? (defaults to inventory)  |
|$user        |Which user to load the items from (defaults to logged in user) |

### Exceptions

|Class thrown           |Description                                                   |
|-----------------------|--------------------------------------------------------------|
|Item_Exception_User    | When no user can be loaded                                   |
|Item_Exception         | A catch-all exception if you just want to catch 1 exception) |

### Examples

Let's say we want to check if the loged in user has 3 items in his inventory, these item's id are 1, 5 and 12:

~~~php
$items = Item::retrieve([1,5,12]);

// let's see if they're found seperately:

if(array_key_exists(1, $items))
{
	//item with id 1 was found
}
if(array_key_exists(5, $items))
{
	//item with id 5 was found
}
if(array_key_exists(1, $items))
{
	//item with id 12 was found
}
~~~

Now let's do it properly and check for errors
~~~php
try {
	$item_ids = [1,5,12];
	$items = Item::retrieve($item_ids);

	if(count($items) == count($item_ids)
	{
		// All items were found
	}
	// Some were found
	else if(count($items) > 0)
	{
		// echo the name of the items that were found
		foreach($items as $id => $item)
		{
			echo $item->item->name . ' was found';
		}
	}
	else
	{
		// No matching items were found in the user's inventory
	}
}
catch(Item_Exception_User $e)
{
	// User wasn't loaded
}
~~~

## How to retrieve items by location

This static helper method prepares a database query for you, you will still need to do the final `->find_all()` on it.
It should give you a chance to pass the query on to a `Paginate` object or manipulate it before executing it.

### Parameters

|Parameter      |Description                                                            |
|---------------|-----------------------------------------------------------------------|
|$location              |Where are should the item be located? (defaults to inventory)  |
|$transferable_check    |Only load items that are transferable? (defaults to FALSE)     |
|$parameter_id          |Specifify a parameter_id value (defaults to null               |
|$user                  |Which user to load the items from                              |

### Exceptions

|Class thrown           |Description                                                   |
|-----------------------|--------------------------------------------------------------|
|Item_Exception_User    | When no user can be loaded                                   |
|Item_Exception         | A catch-all exception if you just want to catch 1 exception) |

### Examples

Let's retrieve all the items from the logged in user's inventory, if we found items we'll loop and display the item's names:

~~~php
$items = Item::location('inventory')
	->find_all();

if(count($items) > 0)
{
	foreach($items as $item)
	{
		echo $item->name() . '<br />';
	}
}
~~~

Now let's do the same, but paginate them with a maximum of 25 items per page:

~~~php
$items = Item::location('inventory');

$paginate = Paginate::factory($items, ['total_items' => 25])
	->execute();

$pagination_menu = $paginate->render();
$list = $paginate->result();

// Print the pagination menu
echo $pagination . '<br />';

// Echo the item names
if(count($items) > 0)
{
	foreach($items as $item)
	{
		echo $item->name() . '<br />';
	}
}
~~~

Let's say we want to create a trade and show all items that the user has in his inventory, we'd only want to show
items that are transferable:

~~~php
$items = Item::location('inventory', TRUE)
	->find_all();

if(count($items) > 0)
{
	foreach($items as $item)
	{
		echo $item->name() . '<br />';
	}
}
~~~

Now let's say we have a trade and we want to show all the items in a lot.
These would all be located in `trade.lot`, all items in trade lots are located here, parameter_id specifies in which trade they would be located.

Let's locate all items for the trade lot with 1 as its id (these items should be transferable of course):

~~~php
$items = Item::location('trade.lot', TRUE, 1)
	->find_all();

if(count($items) > 0)
{
	foreach($items as $item)
	{
		echo $item->name() . '<br />';
	}
}
~~~
