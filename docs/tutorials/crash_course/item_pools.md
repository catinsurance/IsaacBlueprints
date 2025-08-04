---
article: Item Pools
authors: benevolusgoat
comments: true
tags:
    - Tutorial
    - Beginner friendly
    - No Lua
    - XML
    - Repentance
    - Repentance+
    - REPENTOGON
---

{% include-markdown "hidden/crash_course_toc.md" start="<!-- start -->" end="<!-- end -->" %}

Item Pools are the primary method of finding and obtaining your custom collectibles within a run.

## Video tutorial
(This tutorial covers both Item Pools and Active Items)
[![Active Items and Item Pools | Youtube Tutorial](https://img.youtube.com/vi/MwsdlW7ZyQ8/0.jpg)](https://youtu.be/MwsdlW7ZyQ8 "Video tutorial")

## itempools.xml
Once you have [items defined in an `items.xml` file](../crash_course/passive_item.md) within your mod's `content` folder, you will need to create an [itempools.xml](https://wofsauge.github.io/IsaacDocs/rep/xml/itempools.html) file in the same folder in order to add your collectibles into various item pools.

```XML
<ItemPools>
	<Pool Name="treasure">
		<Item Name="My Item" Weight="1" DecreaseBy="1" RemoveOn="0.1"/>
	</Pool>
</ItemPools>
```

For every XML file, there is a "root tag" that goes from the start of the file to the end of the file, and multiple "child tags" that act as the individual entries. In this instance, `itempools.xml` entries will start with `<ItemPools>` and end with `</ItemPools>`. Individual item pool entries start and end with the `Pool` tag, and collectibles within those pools use the `Item` tag. Below are explanations of each variable contained within each of these tags.

The `ItemPools` tag does not contain any variables.

The `Pool` tag only requires one variable: `Name`. It is used to define the item pool the items will be added to.

???- info "`Item` tag variables"
	| Variable Name | Possible Values | Description |
	|:--|:--|:--|
	| Id | int | *(Optional)*<br> The id of the item in the itempool<br> When using this variable, you can't use the "name" variable.|
	| Name | String | *(Optional, recommended)* The name of the item in the itempool.<br> When using this variable, you can't use the "id" variable. |
	| Weight | float | Relative "likelyhood" that this item can be drawn from the pool. Default is `1`. If this value reaches the "RemoveOn" value, the item will no longer be drawn from the pool.|
	| DecreaseBy | float | Value on how often the item can be drawn from the pool. Default is `1`<br>Everytime an item is drawn from the pool, this value is substracted from its `Weight`. This makes the item appear less likely on reroll until the weight reaches the `RemoveOn` value.|
	| RemoveOn | float | If the `Weight` value reaches this, the item is no longer able to be drawn from the pool. Default is `0.1`.|

## Available pools
There is a pre-defined list of item pools you can add your items to. You cannot define your own custom item pools without the use of :modding-repentogon: REPENTOGON. Below is the list of item pool names you can define to add your items to:

???- info "Item pool names"
	| Variable Name | Description |
	|:--|:--|
	|treasure|Treasure Room Pool for Normal/Hard Mode|
	|shop|Shop Pool for Normal/Hard Mode|
	|boss|Boss Room Pool for Normal/Hard Mode|
	|devil|Devil Deal Pool for Normal/Hard Mode|
	|angel|Angel Room Pool for Normal/Hard Mode|
	|secret|Secret Room Pool for Normal/Hard Mode|
	|library|Library Pool for Normal/Hard Mode|
	|shellGame|Exclusive Bag of Crafting Item Pool for all modes|
	|goldenChest|Golden Chest Pool for all modes|
	|redChest|Red Chest Pool for all modes|
	|beggar|Beggar Pool for all modes|
	|demonBeggar|Devil Beggar Pool for all modes|
	|curse|Curse Room Pool for Normal/Hard Mode|
	|keyMaster|Key Master Pool for all modes|
	|batteryBum|Battery Bum Pool for all modes|
	|momsChest|Mom's Chest Pool for the unique chest located in Home|
	|greedTreasure|Treasure Room Pool for Greed/Greedier Mode|
	|greedBoss|Boss Room Pool for Greed/Greedier Mode|
	|greedShop|Shop Pool for Greed/Greedier Mode|
	|greedCurse|Curse Room Pool for Greed/Greedier Mode|
	|greedDevil|Devil Deal Pool for Greed/Greedier Mode|
	|greedAngel|Angel Room Pool for Greed/Greedier Mode|
	|greedSecret|Secret Room Pool for Greed/Greedier Mode|
	|craneGame|Crane Game Pool for all modes|
	|ultraSecret|Ultra Secret Room Pool for Normal/Hard Mode|
	|bombBum|Bomb Bum Pool for all modes|
	|planetarium|Planetarium Pool for Normal/Hard Mode|
	|oldChest|Old Chest Pool for the chest spawned by the Isaac's Tomb collectible|
	|babyShop|Baby Shop Pool for shop items while holding the Adoption Papers trinket|
	|woodenChest|Wooden Chest Pool for all modes|
	|rottenBeggar|Rotten Beggar Pool for all modes|

## :modding-repentogon: Custom Item Pools
With REPENTOGON, you can insert any `Name` you wish to register a new item pool. You can list your items as per usual, and can use `Id` instead of `Name` for item entries in order to add vanilla items to your custom pool.

You can fetch the unique ID of your item pool with Lua using [Isaac.GetItemPoolIdByName](https://repentogon.com/Isaac.html#getpoolidbyname). From there, you can treat it like you would any other item pool ID for fetching collectibles.