# Sending Server Enchantments

Custom Enchantments In Datapacks is a good idea in theory, however, implementing this is a bit more troublesome. 
Namely, the client has to be aware of the enchantments that are dynamically loaded. In order to make clients aware of server-side enchantments, we make use of the Plugin Message packet, with a channel of "minecraft:enchantments_list". 
This channel is used bidirectionally, with servers sending out lists of enchantments, and clients responding with acknowledgements. 
This message can be sent at any time, however the vanilla server should only send these messages on client connect, and any time a new enchantment is loaded, an existing one is reloaded, or one is unloaded, through the use of commands, such as /datapack enable|disable and /reload. 

## Clientbound Message

### Enchantment Structure

The Clientbound Enchantment List Message consists of a series of EnchantmentInfos. These EnchantmentInfos are Structures, and are defined as follows

<table>
<tr>
<th>Name</th>
<th>Type</th>
<th>Description</th>
</tr>
</table>
