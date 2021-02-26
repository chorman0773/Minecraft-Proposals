# Non-Vanilla Extensions
In adition to this repository being a Proposal for Mojang, it is also a specification for implementors. 
To assist in implementation, the following rules apply to Non-vanilla Implementations of this specification:
1. Implementors are permitted, and encouraged, to use the minecraft_ext prefix in-place of the minecraft prefix, wherever it appears in these documents.
2. The "nothing" trigger does not need to be implemented.
3. Clients which do not acknowledge the Enchantment List Packet may be rejected, or may have the custom enchantments implemented in a different manner (IE. Transmitting name&level in Item Lore).
