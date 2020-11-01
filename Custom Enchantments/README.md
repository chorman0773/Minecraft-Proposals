

# Proposal for Custom Enchantments #

With the introduction of Datapacks, Map-makers and server operators gained a powerful tool to change the user experience without adding modifications. 
However, even with this, many features still require modifications to implement, and many are non-trivial. Implementing Custom Enchantments is especially difficult, even with modifications (I have experience with this, having written a custom enchantment system). This proposal enables server operators and map-makers to add custom enchantments in vanilla, as well as to easily add custom enchantments to modded servers/clients. 

The proposed syntax is defined in [CustomEnchantments.md](CustomEnchantments.md). 

The proposal has 4 parts:

1. Adding Enchantment Tags to datapacks and defining how these tags can be used
2. Adding additional vanilla item tags, to make creating custom enchantments easier
3. Defining the Concrete, Work In Progress, syntax and structure for creating custom enchantments
4. Moving the primary work of the vanilla enchantments into a datapack and allowing datapack creators to extend the functionality of these enchantments

## Synopsis ##

[CustomEnchantments.md](CustomEnchantments.md): The primary proposal document. 

[vanilla](vanilla): The folder containing the proposed additions to the Vanilla Datapack. 

[NonVanillaExtensions.md](NonVanillaExtensions.md): Extensions which are not required to appear in a vanilla implementation

[SendingServerEnchantments.md](SendingServerEnchantments.md): A mechanism for sending custom enchantments, including those implemented using mods, to clients. 


