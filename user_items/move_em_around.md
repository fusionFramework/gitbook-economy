# Move 'em around

Items don't just stay in one point, some stay in the inventory, some in the user's shop, some go to trades, they can go anywhere.

## Moving the item to another location

To keep track of the item stacks a nifty little method was added called `move` that helps you to easily do this without having to worry too much.

This method returns `false` if you're trying to move more items than the user has,
otherwise it will return the newly created/updated `Model_User_Item`.

### Parameters
|Parameter      |Description                                                                                            |
|--------------|--------------------------------------------------------------------------------------------------------|
|$location     |Where to move the item to (defaults to inventory)                                                       |
|$amount       |How many copies from this stack should be moved (defaults to 1)                                         |
|$single_stack |Should it automatically become a new item stack and not be added to an existing one? (defaults to true) |
|$parameter_id |Should parameter_id be set during the move you should provide an integer                                |

> `$amount` can be filled with '*' to move all copies to the new location.

### Examples

Let's say we currently have the *Apple* loaded that's located in the user's *inventory*, let's move all of them to the user's safe:

~~~php
$item->move('safe', '*');
~~~

Now let's move 5 copies to the user's shop, we'll have to check if it's possible (the user might not have 5 apples)

~~~php
$move = $item->move('shop', 5);

if($move == false)
{
	RD::error('You don\'t have 5 apples to move to your shop');
}
else
{
	RD::success('5 apples were placed in your shop');
}
~~~

What if the user just created an auction and wants 1 apple to be assigned to the auction?
We'll have to assign the auction's id to the item's parameter_id.

This also means that the items should be stored in seperate stacks, since other stacks will be assigned to other auctions.

~~~php
// do the auction creation

$move = $item->move('auction', 1, FALSE, $auction->id);

if($move == false)
{
	RD::error('You don\'t have an apple to put up for auction');
}
else
{
	RD::success('You\'ve successfully created an auction');
}
~~~

## Transfer the item to another user

Items don't always stay in one account, items can get transfered as gifts, after winning an auction or trade,...

Transfering an item is simple with this method, it will only transfer items that aren't locked a user's account.

When creating items in your game's admin you have the option of making items transferable, if you disabled this on an item it won't be able to leave a user's account.

This method returns `false` if you're trying to transfer more items than the user has,
otherwise it will return the newly created/updated `Model_User_Item`.

### Parameters
|Parameter |Description                                                           |
|----------|----------------------------------------------------------------------|
|$user     |A `Model_User` instance of the user you want to send it to            |
|$amount   |How many copies from this stack should be transfered (defaults to 1)  |

> `$amount` can be filled with '*' to transfer all copies to the new user.

### Exceptions

|Class thrown           |Description                                                                                    |
|-----------------------|-------------------------------------|
|Item_Exception         | When the item is not transferable   |

### Examples

Let's move 5 copies of the loaded item to the currently logged in user

~~~php
$item->transfer(Fusion::$user, 5);
~~~

That was easy, but let's make sure the transfer is successful

~~~php
try {
	$transfer = $item->transfer(Fusion::$user, 5);

	// The amount is less than 5
	if($transfer == false)
	{
		RD::error('You don\'t have :item', [':item' => $item->item->name(5)]);
	}
	else
	{
		RD::success(':item was transfered to :user', [':item' => $item->item->name(5), ':user' => Fusion::$user->username]);
	}
}
catch(Item_Exception $e)
{
	// Item is not transferable
	RD::error($e->getMEssage());
}

~~~
