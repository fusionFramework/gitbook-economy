# Item stacks

To save space in the database we don't store user items individually, instead the same items get stored in stacks with an amount (quantity) assigned.

Each stack needs a location assigned (by default that would be inventory, but this might as well be shop, auction, trade or anything you'd like).

Optionally you can assign a value to the `parameter` or `parameter_id` column which you would use as data for that specific location.

## Working with quantity

You can always check how many items are stored in a stack by calling `$item->amount`

if this ever turns to 0, the stack will be deleted automatically.

To modify the amount the `Model_User_Item` class has a method called `amount`, which works like this:


### Parameters

|Parameter |Description                                                                      |
|----------|---------------------------------------------------------------------------------|
|$type     |The value can be '+' or 'add' when increasing, '-' or 'subtract' when decreasing |
|$amount   |How many copies from this stack should be added or substracted (defaults to 1)   |

The method will return `FALSE` if it's unable to add or subtract the amount (if `$amount` isn't a valid number or if `$amount` is more than
there's present), otherwise it will return `TRUE`

### Examples

It can't get simpler than this:

~~~php
// Add 3 copies
$item->amount('+', 3);
$item->amount('add', 3);

// Remove 2 copies
$item->amount('-', 2);
$item->amount('subtract', 2);
~~~

I would, however, advise to check if subtracting works:

~~~php
// Remove 2 copies
$remove = $item->amount('-', 2);

if($remove == false)
{
	// Oops, did not work out as planned
}
~~~


## What are these parameters?

As you might have seen in the database schema and this guide, there are 2 similar columns; `parameter` and `parameter_id`, but what do they mean?

They were added as an extra way for your user items to have an 'identity', while still having them in stacks,
making it easy to remove, add, move or transfer them and provide a place for you to code in specific behaviour.

### parameter_id

parameter_id is used when items are linked to other database tables, if used it should contain an integer.

#### For example: auctions
When a user creates an auction and selects an item it will do 2 things:

1. It will create a new entry in the `user_auctions` table with all the basic info.

2. It will take 1 copy of the item located in the inventory, move it to auction (where it will be a single-item-stack) and assign the auction's id to parameter_id.

~~~php
    $item->move('auction', 1, FALSE, $auction->id);
~~~

This way it's easy to implement an auction search, you would have to search for items with the location set to auction and load the auction by the item's parameter_id.

#### For example: trades
In trades the exact same principle is used, the only difference would be that it's applied on multiple items at once.

I've worked on several projects in the past that tried to solve it by having a huge `trade_lot` with several `item_X` columns, this is a more efficient and modular aprouch.

### parameter

This one is a little bit different, it can hold all kinds of values since it's a text field.

You can store text or serialized arrays in it which the specific location wants to make use of.

A simple example would be the user shops, that script uses this field to store an item's price.

If you wrote a battle script you could store how many times it's been used before deleting it, or store when it was activated for a certain effect to take place.
