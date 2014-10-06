# Use

As with the stores, it's recommended to store an NPC's id in the model of the section of the site that would required it to be loaded as a (belongs_to) relation.

This makes it easy to load your NPC's with the provided functionality.

For example after loading a store:

~~~php
$store = ORM::factory('Store')
    ->where('store.id', '=', 1)
    ->with('npc')
    ->find();

$npc = $store->npc;
~~~

You could always load an NPC like any other model:
~~~php
// load an NPC with 1 as id
$npc = ORM::factory('NPC', 1);
~~~

## Functionality

Once the NPC has been loaded you can start making use of the provided functionality

### Image
Get the URL to the uploaded NPC

~~~php
$npc->img();
~~~


These are stored in your media folder like this:

~~~php
    /m/npc/$npc->type/$npc->name
~~~

### Messages

Each NPC is supposed to have a set of message pre-defined through the admin.

Each type of NPC will have messages for specific purposes.

For example, NPCs with the type 'store' require 4 different type of messages defined:

| Name | Parameters | Use |
| -- | -- | -- |
| introduction | :store_name, :name(NPC's name) | This is what the NPC would say when visiting the store. |
| sale_success | :item, :price | These messages are shown after buying an item. |
| sold_out | :item | This will be shown if an item is already sold out |
| price_low | :item, :price| This will be shown when the users tries to haggle too much. |

It's possible to add multiple messages for each type, for example;
for the `price_low` you could add

1. Nice try, but :price is a tad too little if you want your :item
2. Are you insane? :price is way too little

Now, how would we go around retrieving one of them?

~~~php
$npc->message('price_low');
~~~

This method will return one of the predefined messages for the needed type (at random).

This makes the NPC have a little bit of character instead of always displaying the same, boring, message.
