# Sending Server Enchantments

Custom Enchantments In Datapacks is a good idea in theory, however, implementing this is a bit more troublesome. 
Namely, the client has to be aware of the enchantments that are dynamically loaded. In order to make clients aware of server-side enchantments, we make use of the Plugin Message packet, with a channel of "minecraft:enchantments_list". 
This channel is used bidirectionally, with servers sending out lists of enchantments, and clients responding with acknowledgements. 
This message can be sent at any time, however the vanilla server should only send these messages on client connect, and any time a new enchantment is loaded, an existing one is reloaded, or one is unloaded, through the use of commands, such as /datapack enable|disable and /reload. 

Message Structure is embedded in the Data Field of a Plugin Message Packet (either Clientbound or Serverbound). The Types of fields are as defined by Netty (see <https://wiki.vg/Protocol>).


## Clientbound Message

### Enchantment Structure

The Clientbound Enchantment List Message consists of a series of EnchantmentInfos. These EnchantmentInfos are Structures, and are defined as follows

<table>
	<tr>
		<th>Name</th>
		<th>Type</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>EnchantmentId</td>
		<td>Identifier</td>
		<td>The Id of the enchantment being sent</td>
	</tr>
	<tr>
		<td>EnchantmentName</td>
		<td>Chat</td>
		<td>The Text Component Name of the Enchantment. For enchantments derived from the Custom Enchantment Format, this is the name field of the enchantment</td>
	</tr>
	<tr>
		<td>EnchantmentDescription</td>
		<td>Chat</td>
		<td>The Text Component Description of the Enchantment. For enchantments derived from the Custom Enchantment Format, this is the description field of the enchantment</td>
	</tr>
	<tr>
		<td>IsLevelMeaningless</td>
		<td>Boolean</td>
		<td>True iff Level is meaningless to the enchantment. For Enchantments Derived from the Custom Enchantment Format, this is true iff maxLevel is 1</td>
	</tr>
</table>

#### Hashcode

The Hashcode of EnchantmentInfoList is computed as though by [PkmCom](https://chorman0773.github.io/PkmCom-APL-Library/AbstractProtocol) `[pkmcom.hash.def]` where `EnchantmentInfo` is a Structure type, consisting of the a String field, 2 Json fields, and a Boolean field and uses the standard hashcode algorithm for structure types. 

For exposition only, the PkmCom Structure of `EnchantmentInfo` is, and its hashcode definition is `hashsum(EnchantmentId,EnchantmentName,EnchantmentDescription,IsLevelMeaningless)`.


<table>
	<tr>
		<th>Field</th>
		<th>Type</th>
		<th>Maps to</th>
	</tr>
	<tr>
		<td>EnchantmentId</td>
		<td>Long String</td>
		<td>The String Representation of the EnchantmentId field</td>
	</tr>
	<tr>
		<td>EnchantmentName</td>
		<td>Long Json</td>
		<td>The Json Representation of the EnchantmentName field, where if the top level element is not an object, its wrapped in a Json Object</td>
	</tr>
	<tr>
		<td>EnchantmentDescription</td>
		<td>Long Json</td>
		<td>The Json Representation of the EnchantmentDescription field, where if the top level element is not an object, its wrapped in a Json Object</td>
	</tr>
	<tr>
		<td>IsLevelMeaningless</td>
		<td>Boolean</td>
		<td>Suprisingly, the boolean value with the same bit representation as the IsLevelMeaningless field</td>
	</tr>
</table>


### Packet


<table>
	<tr>
		<th>Field</th>
		<th>Type</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>EnchantmentMessageId</td>
		<td>Byte Enum</td>
		<td>The Id of the Message. The Clientbound Message sends 0x01 (Server Enchantment list)</td>
	</tr>
	<tr>
		<td>NumEnchantments</td>
		<td>Unsigned Short</td>
		<td>The Length of the EnchantmentInfoList Array</td>
	</tr>
	<tr>
		<td>EnchantmentInfoList</td>
		<td>Array of EnchantmentInfo</td>
		<td>The EnchantmentInfo Structures prepared by the server</td>
	</tr>
</table>




## Serverbound Message

As a response to the Clientbound Message, compatible clients are required to send this on this message on the same channel. 

<table>
	<tr>
		<th>Field</th>
		<th>Type</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>EnchantmentMessageId</td>
		<td>Byte Enum</td>
		<td>The Id of the Enchantment Plugin Message. The Serverbound message sends 0x02 (Enchantment List Acknowledge)</td>
	</tr>
	<tr>
		<td>NumEnchantments</td>
		<td>Unsigned Short</td>
		<td>The Number of Enchantments Recieved by the Client</td>
	</tr>
	<tr>
		<td>ListChecksum</td>
		<td>Int</td>
		<td>The hashcode of the EnchantmentInfoList recieved by the client</td>
	</tr>
</table>
