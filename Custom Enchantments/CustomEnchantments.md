# Custom Enchantments #

This document details how custom enchantments are defined (as well as enchantment tags).


## Advancement Predicates ##

To damage_type's, add the following fields

```
(a damage_type)
	...
	is_fall: Checks if the damage was caused by falling or an ender peral
	is_impact: Checks if the damage was caused by flying into a wall too fast
```

## Enchantment Tags ##

Enchantments can have tags. In a datapacks `data/<namespace>/tags/enchantments` folder, enchantment tags can be defined. 

The format of tags is functionally identical to general tag format in datapacks. Read <https://minecraft.gamepedia.com/Tag> for details.  

Like other tags, enchantment tags can refer to other enchantment tags. 
A circular reference is illegal, as is the same enchantment explicitly occuring multiple times in a given enchantment tag (duplicate enchantment names may occur indirectly, and have no effect). 

### Defined Tags ###

I propose to add the following vanilla enchantment tags 


```
minecraft:curse -> Curse Enchantments(minecraft:vanishing_curse and minecraft:binding_curse)
minecraft:sharpness -> Sharpness and mutex with sharpness (minecraft:sharpness, minecraft:smite, minecraft:bane_of_arthropods)
minecraft:protection -> Protection enchantments (minecraft:protection, minecraft:projectile_protection, minecraft:fire_protection, minecraft:blast_protection)
minecraft:thorns -> Thorns type enchantments (presently only contains minecraft:thorns)
minecraft:treasure -> Enchantments which can appear in villager trades and fishing loot, and can be used in loot tables. 
```

### Enchantment Tags in Datapacks ###

Enchantment Tags may appear whenever a list of named enchantments are expected, prefixed with a #. 
These may be used in `mutex_enchantments`, and in `loot_tables` with the `enchant_randomly` function. This is per the proposal. 
Enchantment tags may also refer to other tags in this manner. An enchantment may not declare a circular reference (that is, an enchantment tag which references itself). 


If `enchant_randomly` is with a tag, the tag selectively expands. Specifically, it expands to the set of enchantments that have the tag, and can apply to that item. If an enchantment is already present in the list of enchantments to add, the enchantment is not included in the tag expansion. 


## Item Tags ##

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
<li>minecraft:offhand -> All items in minecraft:ranged_weapon and minecraft:hoe, as well as minecraft:shield, and minecraft:sheers</li>
</ul>




## Custom Enchantment Structure ##

The custom enchantment named as `<namespace>:[path/...]<target>` is located as `data/<namespace>/enchantments/<path/...><target>.json`. 

Each enchantment is a json file with the format defined below. 
If an enchantment is ill-formed it is treated as if it doesn't exist. 
An error message should be logged when an ill-formed enchantment is loaded.  

An enchantment is ill-formed if one of the following conditions is met:

* The enchantment structure cannot be parsed as a strict json file
* The enchantment's set of applicable items is an empty set
* The enchantment is missing `max_level`, `name`, or `applicable_items`
* The enchantment is missing both of `attributes` and `triggers`
* A trigger defined has an invalid trigger name
* A condition defined by a trigger has an invalid condition name
* An effect defined has an invalid effect name
* A quantity which expects an invariant value is neither a constant, the enchantment level,  a composite value composed of such quantities, or a modified quantity. Any quantity that appears anywhere in an enchantment, except in a `condition` or an `effect`, expects an invariant value. 
* A quantity uses a function which does not exist
* Any required field is omitted at any level. 



### Quantities ###

Most numbers may be obtained from various sources. Whenever a value is expected, the value may be given as a literal number or a quantity: 

```
(a quantity):
	-function: The way the quantity is obtained. May be "constant", "level", "modified", "score", "random", "nbt", or "composite". Defaults to "modified" if a quantity is provided in
```

#### Constant Values ####

Form:

```
(a quantity):
	-function:"constant"
	-value:<A number>

(a quantity):<A number>
```

The quantity is obtained from a constant value, given by the value field. Optionally, the quantity object can be omitted, and the constant be given directly in the place of a quantity object.  

#### Enchantment Level ####

Form:

```
(a quantity):
	-function:"level"
```

Obtains the value from the enchantment level. 

#### Composite Values ####

Form:

```
	-function:"composite"
	-operation: Either "add", "multiply", "divide", or "negate"
	-operands: A list of quantities. Exists iff operation is "add" or "multiply", and must contain at least 2 quantities. 
	-operand: A single quantity to negate. Exists iff operation is "negate"
	-dividend: The value to be divided. Exists iff operation is "divide"
	-divisor: The value to divide by. Exists iff operation is "divide"
```

If operation is `add`, then given that the operands are given by the list `[q1,q2,...,qN]`, then each of q1, q2, ..., qN are evaluated in order. The value of the quantity is `q1 + q2 + ... + qN`

If operation is `multiply`, the operands are given and evaluated as per `add`. The value of the quantity is `q1 * q2 * ... * qN`. If any quantity evaluates to 0, then no subsequent quantity is evaulated, and the value of the quantity is 0.

If operation is `negate`, then the operand, given as `q`, is evaluated. The value of the quantity is `-q`.

If operation is `divide`, then the dividend, given as `q`, is evaluated, followed by the divisor, given by `d`. The value of the quantity is `q/d`. `d` may not evaluate to 0. 

#### NBT Tags ####

Form:

```
(a quantity):
	-function:"nbt"
	-source:Either an entity selector, a co-ordinate set, or "item". provided in a string. 
	-path: The nbt path to search. May not result in more than one value, and must result in a number. 
```


Obtains a value from an NBT Tag. 
The tag can either be from an entity (if source is an entity selector), for a block (if source is a co-ordinate set), or for the item the enchantment is applied to(if source is exactly "item"). 

#### Scores ####

Form:

```
(a quantity):
	-function:"score"
	-source: An entity selector string. 
	-score: The score to check
```

Obtains a value from the scoreboard. 

Score must be the name of an objective. 
Source does not have to have a value in score, be online, or even exist. 
If Source does not have a value in score, then the value is 0. 


#### Modified Quantity ####

Form:

```
(a quantity):
	-function:"modified" (or omitted)
	-base: A constant value defining the value at level 1.
	-modifier: The ammount the quantity is increased by for each level above 1.
```

The quantity `{"function":"modified","base":n,"modifier":m}` is equivalent to 

```
{
	"function":"composite",
	"operation":"add",
	"operands":[
		{"function":"constant","value":n},	
		{
			"function":"composite",
			"operation":"multiply",
			"operands":[
				{"function":"constant","value":m},
				{
					"function":"composite",
					"operation":"add",
					"operands":[
						{"function":"level"},
						{"function":"constant,"value":-1}
					]
				}
			]
		}
	]
}
```

This is provided to make it easier to define the function m(x-1)+b, for m is the modifier, and b is the base. 

#### Random Values ####

```
(a quantity):
	-function:"random"
	-min: The minimum value, inclusive. Optional, defaults to 0
	-max: The maximum value, exclusive. Optional, defaults to 1
```

Generates a random number in [`min`,`max`). 

#### Handling Integers vs. Real Numbers
All values considered here assume that the result is a real number. However, some places quantities are used expects integers. If the result of a quantity is not an integer where one is expected, then the value given is the result after discarding the fractional component. 

### Enchantment Structure ###

Enchantments would use the following structure:

```
(root)
    -name: The raw name string displayed to the client, or the name text component for formatting. 
    -max_level: The maximum level for the enchantment.
    -max_table_level: The maximum level that can be generated from an enchanting table. Optional, if undefined or 0, the enchantment cannot be added from an enchanting table. 
    -applicable_items: The items that the enchantment can be added to from an anvil/enchant_randomly. Can contain tags using tag notation. If the tags cannot be located, they are treated as empty. Required field. If the resultant set is empty, then the enchantment is ill-formed.
    -enchantable_items: The items that can have the enchantment added to directly. Optional. If undefined, defaults to applicable_items. Only has meaning if max_table_level is at least 1.
    -description: The description string/text component. Has no meaning presently.
    -enchantment_weight: The weight to add this enchantment to an item, either from an enchanting table, or with the enchant_randomly loot table function. Defaults to 1
    -mutex_enchantments: The set of enchantments which the enchantment cannot be applied along side of. Defaults to an empty set. 
    -enchantment_level_slot: The level slot the enchantment is in when generating enchantments. Modified quantity, but may be given as an absolute quantity. If given as an absolute quantity, the modifier is treated as 1 and the function is treated as linear. The default is the modified quantity given by {"base":7,"modifier":1}. 
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
	-repair_cost: The ammount to increase the repair cost of enchanted items by. Optional. Defaults to the quantity {"base":1,"modifier":1}. 
	-applicable_slots: The set of slots where the enchantment triggers can be signalled in, and where the attribute modifiers apply in. Optional. If undefined or an empty array, computes the slot as specified below. Each value may be a slot name (see below).
	-weight: The weight of the enchantment at a particular level, for both enchanting tables, and for "enchant_randomly" tags.
	-quality: The weight modifier for "enchant_randomly" tags. May not exist
```

### Enchanting Level Slots ###

In this proposal, enchantment tables now operate on slots, rather than levels. At the gameplay level, there is no apparent difference, the change is only to make interacting with enchantment tables easier. By default, the Enchanting level slot is 7 and increases by 1 per level. The maximum slot is 17. The "Enchantment Table selections" are built by taking an enchantment at Enchanting level slot, then with a 67% chance, adding an additional enchantment at the slot-2, until a slot based maximum number of enchantments is added, no enchantment is added, or the enchantment to be added cannot be added in conjunction with the other enchantments. It will only pick from enchantments applicable to the item. Enchantments with slots greater than 17 or less than 0 cannot be added in an enchanting table. 

<table>
	<tr>
		<th>Slot(s)</th>
		<th>Enchanting Levels</th>
		<th>Max Enchantments</th>
	</tr>
	<tr>
		<td>0-2</td>
		<td>1-4</td>
		<td>1</td>
	</tr>
	<tr>
		<td>3-5</td>
		<td>3-7</td>
		<td>2</td>
	</tr>
	<tr>
		<td>6-8</td>
		<td>6-10</td>
		<td>2</td>
	</tr>
	<tr>
		<td>9-10</td>
		<td>10-15</td>
		<td>4</td>
	</tr>
	<tr>
		<td>11-12</td>
		<td>13-18</td>
		<td>4</td>
	</tr>
	<tr>
		<td>13-14</td>
		<td>15-20</td>
		<td>5</td>
	</tr>
	<tr>
		<td>15</td>
		<td>17-23</td>
		<td>5</td>
	</tr>
	<tr>
		<td>16</td>
		<td>21-29</td>
		<td>6</td>
	</tr>
	<tr>
		<td>17</td>
		<td>30</td>
		<td>6</td>
	</tr>
</table>



### Applicable Slots ###

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
<li>If the item does not have minecraft:wearable, then the set includes the mainhand</li>
<li>If the item has minecraft:offhand, then the set includes the offhand</li>
<li>If the item has minecraft:headgear, then the set includes the helmet slot</li>
<li>If the item has minecraft:chest_wearable, then the set includes the chestplate slot</li>
<li>If the item has minecraft:leggings, then the set includes the legs slot</li>
<li>If the item has minecraft:boots, then the set includes the boots slot</li>
</ul>




## Triggers and Effects ##

Enchantments define a series of Triggers, or events to subscribe to, and effects which apply when the event is triggered. 
Triggers apply whenever the given event is raised, the enchanted item is in an applicable slot, and each condition is met. 
Conditions are applied in short-circuit order as a conjunction, once a single condition fails, no other conditions are evaluated. 
If a condition is not applicable for a given trigger, it is considered to always fail. It is unspecified whether other, previous conditions apply before such conditions are hit, but these conditions are never evaluated. 

### Ordering of Triggers, Effects, and Conditions ###

The ordering of various triggers may be important to the resolution of those triggers. For this reason the ordering of each trigger and effect is well defined. 

The ordering system defines 4 sequence levels: item, enchantment, trigger, and effect 

<ol type="1">
<li>The item sequence level is the set of enchantments on a particular item.</li>
<li>The enchantment sequence level is a particular enchantment on the item</li>
<li>The trigger sequence level is a particular trigger of that enchantment</li>
<li>The effect sequence level is a particular condition or effect of that trigger</li>
</ol>

Each level, except for the item level, is sequenced in the declaration order for that level. Effects applying for different items are unsequenced with respect to each other, regardless of sequence order. 

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

Assassinate is a custom enchantment written using this proposal, that has a 1 in 65536 chance of instantly killing the target bypassing armor and magic, but not invulnerability(using the `damage_target` effect), it is not explicitly part of this Proposal, it is simply included here for exposition purposes.  

With the first sword, the fire aspect enchantment applies first, and sets/adds fire ticks to the target. Then the Assassinate effect applies. If the Assassinate effect does activate, the entity will be on fire (unless it was immune), which may change resultant loot. 

With the second sword, the Assassinate effect applies first. If it does activate, then the target entity will be killed. Then when Fire Aspect evalutates, it fails because the target entity is dead. This means that if the Assassinate effect does activate, the entity will not be on fire, meaning that the resultant loot may change. 

### Global Conditions ###

These conditions may appear in any trigger, and always have meaning. These usually do not check a particular entity, or check the user entity (the entity with the enchanted item). 

#### Conjunction

Joins a set of subconditions with the logical `and` operator. Passes if each chained condition passes. The chained conditions are evaluated in fail-fast short-circuit manner. As soon as one chained condition fails, no other conditions are evaluated, and the conjunction condition fails.

```
(a condition)
    -condition:"conjunction"
    -chained: A list of subconditions 
    	-(a condition)
```

#### Disjunction

Joins a set of subconditions with the logical `or` operator. Passes if any chained condition passes. The chained conditions are evaluated in a succeed-fast short-circuit manner. As soon as one chained condition succeeds, no other conditions are evaluated, and the disjunction succeeds. 

```
(a condition)
	-condition:"disjunction"
	-chained: A list of subconditions.
		-(a condition)
```

#### Negation

Applies the logical `negation` operator to a chained subcondition. Passes if the chained subcondition fails. The direct chained condition may not be a `negation` condition.  

```
(a condition)
	-condition:"negation"
	-chained: (a condition)
```

#### Random Chance

Passes randomly based on a biased chance. If the chance is at least 1, the condition is not evaluated, and succeeds unconditionally. The chance may not be less than 0.

```
(a condition)
	-condition:"random_chance"
	-chained: A quantity which results in a real number in (0,1]. If a random number is less than this value, then the condition passes. 
```

#### First Trigger

Triggers if this particular event has not be handled by the same enchantment on a different item. 

```
(a condition)
	-condition:"first_trigger"
```


#### In Biome

Checks if the biome the user is in meets certain criteria

```
(a condition)
	-condition:"in_biome"
	-biome: Json Object containing various tags to match. All defined tags must match for the condition to pass.
		-temperature_range: The temperature range the biome must fall in
			-min: The minimum temperature. Defaults to -Infinity
			-max: The maximum temperature. Defaults to Infinity
		-biome_name: Checks if the biome is either a specific biome or one of a list of biomes.
		-has_rain: Checks if the biome can rain or cannot.
		-has_snow: Checks if the biome can snow or cannot.
```

#### Weather

Checks the current weather

```
(a condition)
	-condition:"weather"
	-weather: The weather that must be occuring, or a list of such. If a list is provided, passes if the weather is any of those weathers. Acceptable values are "rain", "storm", and "clear". 
```

#### In Block

Checks if the user is in a particular type of block. Specifically checks if at least 1 of the 8 blocks the user can be inside of matches

```
(a condition)
	-condition:"in_block"
	-block_state: The block state to match against. Same as similar block_state objects used by advancement criteria
```

#### Predicate

Checks if the user passes a particular predicate.

```
(a condition)
        -condition:"predicate"
	-predicate: The name of the predicate to evaluate
```

### Global Effects

These effects, like the global conditions, can be applied regardles of the trigger being used. These usually only apply to the user. 

#### Add User Status

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

#### Heal User

Heals the user by a particular ammount. Undead entities recieve \*1.5 damage instead.

```
(an effect)
	-effect:"heal_user"
	-amount: The amount (in half hearts) to heal the user by. Can be absolute or modified
```

#### Harm User

Inflicts an amount of damage to the user. Undead entities heal by \*0.67 instead, unless the entity is immune.

```
(an effect)
	-effect:"harm_user"
	-amount: The ammount of damage to deal. Can be absolute or modified.
	-damage_type: The type of damage dealt by the Enchantment. Defaults to the magic damage dealt by harming potions
		-death_message: The death message to display when a player dies to the damage
		-bypasses_armor: True if the damage is not reduced by armor. Defaults to false
		-bypasses_magic: True if the damage is not reduced by enchantments. Defaults to false
		-bypasses_invulnerability: True if the damage can be dealt to invulnerable targets. Defaults to false
		-is_fire: True if this is fire damage. Defaults to false
		-is_explosion: True if this explosion damage. Defaults to false
		-is_magic: True if this is magic damage. Defaults to true
		-is_projectile: True if this is projectile damage. Defaults to false.
```

#### Damage User

Inflicts damage of a particular type to the user. 

```
(an effect)
	-effect:"damage_user"
	-amount: The amount of damage to deal. Can be absolute or modified
	-damage_type: The type of damage to deal.
		-death_message: The death message to display when a player dies to the damage
		-bypasses_armor: True if the damage is not reduced by armor. Defaults to false
		-bypasses_magic: True if the damage is not reduced by enchantments. Defaults to false
		-bypasses_invulnerability: True if the damage can be dealt to invulnerable targets. Defaults to false
		-is_fire: True if this is fire damage. Defaults to false
		-is_explosion: True if this explosion damage. Defaults to false
		-is_magic: True if this is magic damage. Defaults to false
		-is_projectile: True if this is projectile damage. Defaults to false.
```

#### Ignite User \\

Sets or increases the fire ticks of the user, unless the user is immune to fire. 

```
(an effect)
	-effect:"ignite_user"
	-time: The number of fire ticks to add to the user. Can be absolute or modified
```

#### Strike User

If the user is exposed to the sky, strikes the user with lightning. 

```
(an effect)
	effect:"strike_user"
```

#### Run Function

Runs a given function as though the server executed execute as *user* *position (see below)* run function *function*

```
(an effect)
	effect:"run_function"
	function:A function name or tag
```

*user* is an entity selector or entity name which names the user entity. 

*position* depends on the function trigger:
* for `entity_attacked` and `user_attacked` triggers, *position* is given to be `at ` *entity*, where *entity* is the target for `entity_attacked` and source for `user_attacked` triggers. 
* for `projectile_landed` and `block_broken` triggers, *position* is given to be `positioned at` *pos* where pos the position of the projectile or broken block respectively. 
* for any other trigger, *position* is `at` *user*.

Notes:
* It is intended that the function be run with the maximum permissions possible for the current server software. In the case of the vanilla server, or vanilla permissions, this means the command is run with permission level 4 (reguardless of operatorPermissionLevel or the permission level of *user*). 
* The execute command is for exposition only.  It is unspecified how this function is run, but the command must be run as the user, and positioned as above. 
* The command chain max applies to the function as a separate chain, even if the trigger applied as the result of a command.  

#### Repair Item

Repairs the item affected by the enchantment. 

```
(an effect):
	-effect:"repair_item"
	-value: A quantity which resolves to an integer number of durability points to repair the item by. May not be negative
```

#### Damage Item

Damages the item affected by the enchantment

```
(an effect):
	-effect:"damage_item"
	-value: A quantity which resolves to an integer number of durability points to damage the item by. May not be negative.
```


### Null Trigger

Event is never fired. Can be useful for writing vanilla enchantments that require internal support, as all enchantments defined using this proposal require either at least one trigger, or at least one attribute modifier

```
(a trigger)
	-trigger:"null"
```

### Tick Trigger

Event fired 20 times per second, or after a set period of time

```
(a trigger)
	-trigger:"tick"
	-delay: The number of ticks before the next event is fired. Optional, defaults to 1.
```


### Item Equipped

Event fired when an item with the enchantment is added to an applicable item slot, except from another applicable item slot of the same entity.  

```
(a trigger)
	-trigger:"equipped"
```

### Item Unequipped

Event fired when the item with the enchantment is removed from an applicable item slot, unless it is added to another applicable item slot of the same entity. 

```
(a trigger)
	-trigger:"unequipped"
```

### Entity Attacked

Fired when the user deals damage to an entity. 

```
(a trigger)
	-trigger:"entity_attacked"
```

#### Applicable Conditions

The following conditions apply to `entity_attacked` triggers.

##### Target Entity

Checks the entity that was dealt damage. 

```
(a condition)
	-condition:"entity_data"
	-entity: The Json representation of an entity. Same as similar fields in enchantment criteria. 
```

##### Target Entity Type

Checks the entity type of the damaged entity.

```
(a condition)
	-condition:"entity_type"
	-entity_types: A list of entity types. Can contain any number of entity tags, without the # prefix.
```

#### Applicable Effects

The following effects can be applied from `entity_attacked` triggers

##### Heal Target

Same as similar `heal_user` effect, except applies to the attacked entity rather then the user.  

```
(an effect)
	-effect:"heal_target"
	...
```

##### Harm Target

Same as similar `harm_user` effect, except applies to the attacked entity rather then the user. 

```
(an effect)
	-effect:"harm_target"
	...
```

##### Damage Target

Same as similar `damage_user` effect, except applies to the attacked entity rather then the user.

```
(an effect)
	-effect:"damage_target"
	...
```

##### Add Damage

Modifies the damage dealt by an additive factor

```
(an effect)
	-effect:"add_damage"
	-amount: The amount to increase the damage by. Can be absolute or modified.
```

##### Modify Damage

Modifies the damage dealt by a multiplicative factor

```
(an effect)
	-effect:"modify_damage"
	-modifier: The ammount to modify the damage by. Can be absolute or modified.
```

##### Ignite Target

Adds an amount of fire ticks to the target of the attack.  Similar to the `ignite_user` effect.

```
(an effect)
	-effect:"ignite_target"
	...
```

##### Strike Target

If the target is exposed to the sky, strikes the target with lightning. Similar to the `strike_user` effect.

```
(an effect)
	-effect:"strike_target"
	...
```


### User Damaged

Fired whenever a source deals damage to the user.

```
(a trigger)
	-trigger:"user_damaged"
```

#### Applicable Conditions

The following conditions can be applied from `user_damaged` triggers. 

##### Damage Source

Checks the source of the damage

```
(a condition)
	-condition:"damage_source"
	-entity: The entity which was the original source of the damage. Same as similar entity tags used in advancement criteria.
	-direct_entity: The entity which actually dealt the damage (the projectile if that was the type of damage). Same as similar entity tags used in advancement criteria.
	-damage_type: The type of damage dealt. Same as similar damage_type tags used in advancement criteria.
```

#### Applicable Effects

##### Add Damage

The `add_damage` effect applies to `user_damaged` triggers.

##### Modify Damage

The `modify_damage` effect applies to `user_damaged` triggers.

##### Reduce Damage

Applies a protection effect based on a protection factor. This may be more useful then `modify_damage` in most circumstances. 
The Enchantment Protection Factor (EPF) is used as specified at <https://minecraft.gamepedia.com/Armor#Enchantments>.

```
(an effect)
	-effect:"reduce_damage"
	-factor: An absolute or modified quantity that modifies the EPF
```


### User Attacked

Fired whenever an entity attacks the user (a `user_damaged` trigger will apply at some indeterminate time in relation to this trigger)   

```
(a trigger)
	-trigger:"user_attacked"
```

#### Applicable Conditions

##### Source Entity

The `entity_data` condition applies to `user_attacked` triggers. Checks the source entity rather then the target entity.

##### Source Entity Type

The `entity_type` condition applies to `user_attacked` triggers. Checks the source entity rather then the target entity.

##### Damage Source

The `damage_source` condition applies to `user_attacked` triggers. 

#### Applicable Effects

##### Heal/Harm/Damage Source

Heals/Harms/Damages the source entity by a given amount. Similar to the `heal_user`, `harm_user`, and `damage_user` effects respectively.

Heal Source:

```
(an effect)
	-effect:"heal_source"
	...
```

Harm Source:

```
(an effect)
	-effect:"harm_source"
	...
```

Damage Source:

```
(an effect)
	-effect:"damage_source"
	...
```

##### Ignite/Strike Source

Ignites/Strikes the source with lighting. Similar to the `ignite_user` and `strike_user` effects, respectively.

Ignite Source:

```
(an effect)
	-effect:"ignite_source"
	...
```

Strike Source:

```
(an effect)
	-effect:"strike_source"
	...
```

### Projectile Fired

Applies when a projectile weapon or trident fires a projectile. There are no additional conditions associated with this trigger

```
(a trigger)
	-trigger:"projectile_fired"
	...
```

#### Applicable Effects

##### Modify Power

Modifies the power of the projectile, 

```
(an effect)
	-effect:"modify_power"
	-modifier: An absolute or modified quantity to add to the power tag of the projectile
```

##### Ignite Projectile

Same as `ignite_user`, `ignite_target`, and `ignite_source` effects, but applies to the projectile. 

```
(an effect)
	-effect:"ignite_projectile"
	...
```

### Item Damaged

Applies whenever the item is damaged. This applies individually for each point of damage. 

```
(a trigger):
	-trigger:"item_damaged"
	...
```

#### Applicable Conditions

##### Item

Checks the item that was damaged, before it was damaged. 

```
(a condition):
	-condition:"item"
	-item: An item predicate. Same as similar predicates for advancements.
```

#### Applicable Effects

##### Negate Damage

Stops the item from losing durability

```
(an effect):
	-effect:"negate_damage"
```

## Vanilla Datapack Additions

The following files are proposed to be added to the vanilla datapack, to transition from the old system to the system applicable in this document (see the vanilla subfolder):

```
(root)
	/minecraft
		/enchantments
			/sharpness.json
			/smite.json
			/bane_of_arthopods.json
			/protection.json
			/projectile_protection.json
			/blast_protection.json
			/fire_protection.json
			/feather_falling.json
			/efficiency.json
			/silk_touch.json
			/fortune.json
			/looting.json
			/fire_aspect.json
			/sweeping_edge.json
			/aqua_affinity.json
			/respiration.json
			/power.json
			/punch.json
			/infinity.json
			/flame.json
			/mending.json
			/piercing.json
			/multishot.json
			/quickdraw.json
			/frost_walker.json
			/depth_strider.json
			/lure.json
			/luck_of_the_sea.json
			/loyalty.json
			/impaling.json
			/riptide.json
			/channelling.json
			/thorns.json
			/unbreaking.json
			/vanishing_curse.json
			/binding_curse.json
		/tags
			/enchantments
				/curse.json
				/protection.json
				/sharpness.json
				/thorns.json
				/treasure.json
			/entity_types
				/undead.json
				/aquatic.json
				/arthropod.json
			/items
				/sword.json
				/projectile_weapon.json
				/ranged_weapon.json
				/weapon.json
				/pickaxe.json
				/axe.json
				/shovel.json
				/hoe.json
				/tool.json
				/offhand.json
				/helmet.json
				/head_wearable.json
				/chestplate.json
				/chest_wearable.json
				/leggings.json
				/boots.json
				/armor.json
				/wearable.json
				/damagable.json
```

