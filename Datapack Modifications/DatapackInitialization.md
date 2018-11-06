<h1>Datapack Initialization</h1>

This proposal adds the ability to set "one time initialization" functions for a datapack, as well as depend on other datapacks. 





<h3>Datapack Initializers</h3>

Minecraft already provides the ability to check when a datapack is loaded. 
However certain initialization really should only be performed once per world load. 
In addition to the `minecraft:load` tag, I propose the `<namespace>:init` tag. This vanilla tag is placed in generic namespaces so as to associate datapack initialization with a particular namespace. Note that multiple datapacks can have multiple namespaces, and each namespace is considered its own datapack as far as the initialization system is concerned. 

Whenever datapacks are (re)loaded, before functions with `minecraft:load` are called, if any functions tagged with `<namespace>:init` are found, and the datapack `<namespace>` has not been initialized, then all functions with that tag are called, then that datapack is initialized. 
Different datapacks registering initialization functions are called in an unspecified order.




