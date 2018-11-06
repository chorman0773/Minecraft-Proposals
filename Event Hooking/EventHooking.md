#Hooking Events with functions

This proposal adds the ability to hook various events to run a list of functions with a particular tag. 




##How to hook events
Events are bound to a set of function tags.

When a given event is raised all functions tagged with the appropriate event tag are executed in a particular manner, 
depending on the kind of event. 

All events are executed with elevated permissions, at and as the appropriate entity (if applicable)

##Events

These are the events that are hookable by functions in this proposal

###Datapack Initialization

Minecraft already provides the ability to check when a datapack is loaded. 
However certain initialization really should only be performed once per world load. 
In addition to the `minecraft:load` tag, I propose the `<namespace>:init` tag. This vanilla tag is placed in generic namespaces so as to associate datapack initialization with a particular namespace. Note that multiple datapacks can have multiple namespaces, and each namespace is considered its own datapack as far as the initialization system is concerned. 

Whenever datapacks are (re)loaded, before functions with `minecraft:load` are called, if any functions tagged with `<namespace>:init` are found, and the datapack `<namespace>` has not been initialized, then all functions with that tag are called, then that datapack is initialized. 
Different datapacks registering initialization functions are called in an unspecified order.




