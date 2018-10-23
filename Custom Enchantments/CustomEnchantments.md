<h1>Custom Enchantments</h1>

This document details how custom enchantments are defined (as well as enchantment tags).



<h2>Enchantment Tags</h2>

Enchantments can have tags. In a datapacks `data/<namespace>/tags/enchantments` folder, enchantment tags can be defined. 

The format of tags is functionally identical to general tag format in datapacks. Read <https://minecraft.gamepedia.com/Tag> for details.  

Like other tags, enchantment tags can refer to other enchantment tags. 
A circular reference is illegal, as is the same enchantment explicitly occuring multiple times in a given enchantment tag (duplicate enchantment names may occur indirectly, and have no effect). 

<h3>Defined Tags</h3>

I propose to add the following vanilla enchantment tag 


```
minecraft:curse
```

<h3>Enchantment Tags in Datapacks</h3>

Enchantment Tags may appear whenever a list of named enchantments are expected, prefixed with a #. 
These may be used in `mutex_enchantments`, and in `loot_tables` with the `enchant_randomly` function. This is per the proposal. 
Enchantment tags may also refer to other tags in this manner. An enchantment may not declare a circular reference (that is, and enchantment tag which references itself). 


If `enchant_randomly` is with a tag, the tag selectively expands. Specifically, it expands to the set of enchantments that have the tag, and can apply to that item. If an enchantment is already present in the list of enchantments to add, the enchantment is not included in the tag expansion. 


<h2>Item Tags</h2>

The following new item tags can be added, to make creating custom enchantments easier. 

<ul>
<li>minecraft:sword -> All Swords</li>
<li>minecraft:projectile_weapon -> Presently just minecraft:bow</li>
<li>minecraft:ranged_weapon -> All items in minecraft:projectile_weapon as well as minecraft:trident</li>
<li>minecraft:weapon -> All items in minecraft:sword and minecraft:ranged_weapon</li>
<li>minecraft:helmet -> All helmets</li>
<li>minecraft:head -> All head items</li>
<li>minecraft:headgear -> All items in minecraft:helmet, as well as all heads/skulls and minecraft:carved_pumpkin</li>
<li>minecraft:chestplate -> All Chestplates</li>
<li>minecraft:chest_wearable -> All items in minecraft:chestplate as well as minecraft:elytra</li>
<li>minecraft:leggings -> All Leggings</li>
<li>minecraft:boots -> All Boots</li>
<li>minecraft:armor ->  All Armor</li>
<li>minecraft:wearable -> All Wearable items. Specifically all items in minecraft:armor, minecraft:chest_wearable, and minecraft:headgear</li>
<li>minecraft:pickaxe -> All Pickaxes</li>
<li>minecraft:axe -> All Axes</li>
<li>minecraft:shovel -> All Shovels</li>
<li>minecraft:hoe -> All hoes</li>
<li>minecraft:tool -> All Tools that can be damaged. Specifically all items in minecraft:pickaxe, minecraft:axe, minecraft:shovel, minecraft:hoe, as well as minecraft:shears</li>
<li>minecraft:damagable -> All items that can be damaged. Specifically all items in minecraft:armor, minecraft:tool, and minecraft:shield</li>
<li>minecraft:offhand -> All items in minecraft:ranged_weapon and 
</ul>




<h2>Custom Enchantment Structure</h2>

The custom enchantment named as `<namespace>:[path/...]<target>` is located as `data/<namespace>/enchantments/<path/...><target>.json`. 

Each enchantment is a json file with the format defined below. 
If an enchantment is ill-formed it is treated as if it doesn't exist. 

An enchantment is ill-formed if one of the following conditions is met:
<ul>
<li>The enchantment structure cannot be parsed as a strict json file</li>
<li>The enchantment's set of applicable items is an empty set</li>
<li>The enchantment is missing `max_level`, `name`, or `applicable_items`</li>
<li>The enchantment is missing both of `attributes` and `triggers`</li>
<li>An effect refers to a trigger condition that does not exist</li>
<li>A trigger defined has an invalid trigger name</li> 
<li>A condition defined by a trigger has an invalid condition name</li>
<li>An effect defined has an invalid effect name</li>
</ul>

<h3>Absolute and Modified Quantities</h3>

Most numbers in enchantment effects and conditions can be either absolute or modified. 

Absolute quantities are just json numbers. These quantities are not affected by the enchantment level. 

Modified quantities are special json structures. These quantities are affected by the enchantments level. A modified quantity with a `max_level` field set to 1 is useless, and can be treated as an absolute quantity with the given `base` field. 


Modified quantities use the following format

```
(a quantity)
	-base: The value of the quantity, with the enchantment level at 1.
	-modifier: The value the quantity is increased by for each level above 1.
```

Modified quantities are linear functions of x being the level of the enchantment. 
Given:
<ul>
<li>x, the level of the enchantment, at least 1</li>
<li>b, the base value of the modified quantity</li>
<li>m, the modifier of the modified quantity</li>
</ul>

One can define the resultant value for the a given level as `f(x)=m(x-1)+b`. 


<h3>Enchantment Structure</h3>

Enchantments would use the following structure:

```
(root)
    -name: The raw name string displayed to the client, or the name text component for formatting. 
    -max_level: The maximum level for the enchantment.
    -max_table_level: The maximum level that can be generated from an enchanting table. Optional, if undefined or 0, the enchantment cannot be added from an enchanting table. 
    -applicable_items: The items that the enchantment can be added to from an anvil/enchant_randomly. Can contain tags using tag notation. If the tags cannot be located, they are treated as empty. Required field. If the resultant set is empty, then the enchantment is ill-formed 
    -enchantable_items: The items that can have the enchantment added to directly. Optional. If undefined, defaults to applicable_items. Only has meaning if max_table_level is at least 1.
    -description: The description string/text component. Has no meaning presently.
    -enchantment_weight: The weight to add this enchantment to an item, either from an enchanting table, or with the enchant_randomly loot table function. Defaults to 1
    -mutex_enchantments: The set of enchantments which the enchantment cannot be applied along side of. Defaults to an empty set. 
    -enchantment_level_slot: The level slot the enchantment is in when generating enchantments. Defaults to 7. The slot is increased by 1 for each level above 1 of the enchantment. Only has meaning if max_table_level is at least 1. See below for the meaning of each slot. 
    -attributes: The list of attribute modifiers to add to the enchanted item
    	-(an attribute modifier)
    		-attribute: The attribute name, like generic.movementSpeed. 
    		-operation: The operation to apply, Defaults to 0 (additive)
    		-value: The value of the enchantment, either an absolute or modified quantity
    -triggers: A named list of triggers. Each trigger subscribes to a particular event
    	-(name):(a trigger)
    		-trigger: The name of the event to subscribe to. See list below
    		-conditions: The list of conditions that must be fulfilled for effects subscribed to this event to apply.
   				-(a condition)
   					-condtion: The condition type. See below
			-effects: A list of conditions to apply when the event is raised
				-(an effect)
					-effect: The effect name. See list below
		-repair_cost: The ammount to increase the repair cost of enchanted items by. Can be an absolute or modified quantity. Optional. Defaults to The modified quantity {"base":1,"modifier":1}. 
		-applicable_slots: The set of slots where the enchantment triggers can be signalled in, and where the attribute modifiers apply in. Optional. If undefined or an empty array, computes the slot as specified below. Each value may be a slot name (see below).
```

<h2>Applicable Slots</h2>

Enchantments have applicable slots. 
These slots are computed from their applicable items. 
The set of Applicable Slots for an enchantment is the union of the set of applicable slots of each of its applicable items. 
This may also be explicitly specified, and will override the default set. 

The slot names used are as follows:
<ul> 
<li>mainhand -> The enchantment applies in the mainhand</li>
<li>offhand -> The enchantment applies in the offhand</li>
<li>helmet -> The enchantment applies in the helmet slot</li>
<li>chestplate -> The enchantment applies in the chestplate slot</li>
<li>legs -> The enchantment applies in the leggings slot</li>
<li>boots -> The enchantment applies in the boots slot</li>
</ul>

The set of applicable slots of an item is based on the item tags of that item, using the following rules:
<ul>
<li>If the item does not have minecraft:armor, then the set includes the mainhand</li>
<li>If the item has minecraft:offhand, then the set includes the offhand</li>
<li>If the item has minecraft:headgear, then the set includes the helmet slot</li>
<li>If the item has minecraft:chest_wearable, then the set includes the chestplate slot</li>
<li>If the item has minecraft:leggings, then the set includes the legs slot</li>
<li>If the item has minecraft:boots, then the set includes the boots slot</li>
</ul>




<h2>Triggers and Effects</h2>

Enchantments define a series of Triggers, or events to subscribe to, and effects which apply when the event is triggered. 
Triggers apply whenever the given event is raised, the enchanted item is in an applicable slot, and each condition is met. 
Conditions are applied in short-circuit order as a conjunction, once a single condition fails, no other conditions are evaluated. 
If a condition is not applicable for a given trigger, it is considered to always fail. It is unspecified whether other, previous conditions apply before such conditions are hit, but these conditions are never evaluated. 

<h3>Ordering of Triggers, Effects, and Conditions</h3>

The ordering of various triggers may be important to the resolution of those triggers. For this reason the ordering of each trigger and effect is well defined. 

The ordering system defines 4 sequence levels: item, enchantment, trigger, and effect 

<ol type="1">
<li>The item sequence level is the set of enchantments on a particular item.</li>
<li>The enchantment sequence level is a particular enchantment on the item</li>
<li>The trigger sequence level is a particular trigger of that enchantment</li>
<li>The effect sequence level is a particular condition or effect of that trigger</li>
</ol>

Each level, except for the item level, is sequenced in the sequence order for that level. Effects applying for different items are unsequenced with respect to each other, regaurdless of sequence order. 

The sequence orders are defined as follows:

<ul>
<li>Different enchantments on a particular item are sequenced in the order they appear on the item</li>
<li>Different triggers of the same type for a particular enchantment are sequenced in the order they are declared in the enchantment's structure</li>
<li>Different conditions for a particular trigger are sequenced in the order they are declared, with failing short circuit evaluation (if a condition fails no other conditions are evaluated)</li>
<li>Different effects for a particular trigger are sequenced in the order they are declared, after each condition is evaluated. If a condition fails, no effects are evaluated</li>
</ul>

Sequence order between enchantments at a given level may prevent other effects from occuring. For example, given the following swords:

```
(1)
minecraft:diamond_sword
Enchantments:
Fire Aspect II (minecraft:fire_aspect)
Assassinate (gac14:instant_death_blade)
```

```
(2)
minecraft:diamond_sword
Enchantments:
Assassinate (gac14:instant_death_blade)
Fire Aspect II (minecraft:fire_aspect)
```

With the first sword, the fire aspect enchantment applies first, and sets/adds fire ticks to the target. Then the Assassinate effect applies. If the Assassinate effect does activate, the entity will be on fire (unless it was immune), which may change resultant loot. 

With the second sword, the Assassinate effect applies first. If it does activate, then the target entity will be killed. Then when Fire Aspect evalutates, it fails because the target entity is dead. This means that if the Assassinate effect does activate, the entity will not be on fire, meaning that the resultant loot may change. 

<h3>Global Conditions</h3>

These conditions may appear in any trigger, and always have meaning. These usually do not check a particular entity, or check the user entity (the entity with the enchanted item). 

<h4>Conjunction</h4>

Joins a set of subconditions with the logical `and` operator. Passes if each chained condition passes. The chained conditions are evaluated in fail-fast short-circuit manner. As soon as one chained condition fails, no other conditions are evaluated, and the conjunction condition fails.

```
(a condition)
    -condition:"conjunction"
    -chained: A list of subconditions 
    	-(a condition)
```

<h4>Disjunction</h4>

Joins a set of subconditions with the logical `or` operator. Passes if any chained condition passes. The chained conditions are evaluated in a succeed-fast short-circuit manner. As soon as one chained condition succeeds, no other conditions are evaluated, and the disjunction fails. 

```
(a condition)
	-condition:"disjunction"
	-chained: A list of subconditions.
		-(a condition)
```

<h4>Negation</h4>

Applies the logical `negation` operator to a chained subcondition. Passes if the chained subcondition fails. The direct chained condition may not be a `negation` condition.  

```
(a condition)
	-condition:"negation"
	-chained: (a condition)
```

<h4>Random Chance</h4>

Passes randomly based on a biased chance. If the chance is at least 1, the condition is not evaluated, and succeeds unconditionally. The chance may not be less than 0.

```
(a condition)
	-condition:"random_chance"
	-chained: The real number chance which a random number from 0 inclusive to 1 exclusive, must be at most for the condition to pass. May be an absolute or a modified quantity
```

<h4>First Trigger</h4>

Triggers if this particular event has not be handled by the same enchantment on a different item. 

```
(a condition)
	-condition:"first_trigger"
```

<h3>Global Effects</h3>

These effects, like the global conditions, can be applied regardles of the trigger being used. These usually only apply to the user. 

<h4>Add User Status</h4>

Adds a particular status effect to the user. 

```
(an effect)
	-effect:"add_user_status"
	-status: A status effect name, such as minecraft:haste.
	-amplifier: The effect amplifier. May be either an absolute or a modified quantity. Optional, defaults to 0.
	-duration: The duration of the effect, in ticks. May be either an absolute or a modified quantity. Optional, defaults to 600.
	-replace: If true, replaces any existing potion effects with the same amplifier and status. Optional, defaults to false
	-ambient: Set to true if the effect is ambient (like a Beacon). Optional, defaults to false
	-hide_particles: Set to true if the particles are hidden. Optional, defaults to false.
```






