# Item commands

Commands are actions that are performed when consuming an item in the inventory.

Any item can have one or more commands, the item's type (`default_command`) defines a command that every item of that type is forced
to have, you always have the option of adding extra commands on any item you want.

This default command offers a restriction though, for example, if an item type's default command does not come
from the pet category you won't be able to assign other pet-related commands to related items.

## Type of commands

### General
These commands are default commands that you'd display in the item's initial action screen.

### Move
This kind of command allows the user to move an item around to a different location.

### Pet
This type of command requires a pet to be loaded before it can be executed.

This means that user will have to select a pet before the item can be consumed.

Any assigned pet action will then be executed on the selected pet.

### User

This kind of command will be aplied to the logged in user.


