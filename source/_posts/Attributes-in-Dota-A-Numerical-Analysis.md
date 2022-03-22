---
title: 'Attributes in Dota'
date: 2022-03-22 14:48:40
tags: 随笔
---

A Numerical Analysis

<!--more-->

## Armor

> Armored to the teeth! -- clockwerk

Armor is a stat that reduces (or increases, if negative) physical damage a unit takes from abilities and attacks. Every unit is capable of gaining or losing armor, and most units start with a small amount of base armor, some even starting with negative armor. A hero's armor can be passively increased with Agility attribute symbol.png agility via leveling up, certain items, Talent talents and some abilities. Armor of any unit can also be temporarily increased or reduced with some abilities.

$$\sum Armor = MainArmor + BonusArmor$$

### Base Armor

Base armor is the part of the main armor that never changes throughout a game. It consists of one fixed value set for each unit individually. The base armor of a unit can be a negative number. 

### Main Armor

Main armor consists of base armor and the armor granted by a hero's agility. The only way to improve a hero's main armor is to increase its Agility attribute symbol.png agility, which is gained by leveling up, acquiring certain items, or with the help of certain abilities.

The main armor is defined as

$$MainArmor = (BaseArmor + \sum Agility/6) * (1 - ArmorNegation)$$

### Bonus Armor

Bonus armor is the armor value shown in green numbers with a plus on the left, right after the white armor number on a unit's statistics. Whenever an armor granting item or armor increasing ability shows a +Value Armor.

Probably the most important difference between main and bonus armor is that illusions only benefit from main armor, although their HUD still shows the bonus armor just like on other heroes, to make them less obvious to the enemy.

Sources of multiple bonus armor stack additively.

### Modifying Armor

A lot of abilities and items have abilities that grant or reduce armor.

These changes mostly affect bonus armor, and are never added to the main armor.

### Damage Multiplier

All units, including buildings, have an inherent base armor value and defense class. Any physical damage dealt to a unit is multiplied by the damage multiplier, resulting in either damage reduction or increase depending on the target's armor value.

Damage Multiplier is also used to calculate a unit's Effective HP (EHP). The lower the damage multiplier, the higher the Effective HP of a unit, vice versa.

|Armor Formula Base $b$ |Armor Formula Factor $f$|
|:-:|:-:|
|1|0.06|

For any real-valued armor, the damage multiplier is defined as

$$D^{M} = 1 - \frac{f*Armor}{b+f * |Armor|} \in (0, 2)$$

### Effective HP

$$ EffectiveHP = \frac{CurrentHP}{D^M}$$

The total amount of physical damage a unit can take due to the armor it has is known as Effective HP (or EHP). 16.66 armor adds 100% to EHP. This means a unit with 1000 health and 16.66 armor can take 2000 physical damage. The formula adds 6% EHP per point of armor regardless of the previous armor value.

However, EHP value is of a diminished benefit as a player gaining 1 armor from 0 armor would increase EHP from 100% to 106%, while a player gaining 1 armor from 50 armor would increase EHP from 400% to 406%. On the other hand, the effectiveness of armor is much lower at negative values (refer to the attached graph). Total EHP trends downwards towards 50%, meaning a unit with 1000 health and infinitely low armor would still be able to take 500 physical damage. At this point, losing or gaining 1 armor has essentially no effect.

## Magic Resistance

> The magic harms me not. --Anti-Mage

Magic Resistance is a stat that reduces (or increases, if negative) magical damage a unit takes from spells and magic attacks. Every unit is capable of gaining or losing magic resistance, and most units start with a small amount of base resistance. A hero's magic resistance can be passively increased with Talent talents, certain items, and some abilities. Magic resistance of any unit can also be temporarily increased or reduced with some abilities.

$$MagicResistance = 1 - (1 - BasicResistance) * \prod(1 - Modifier)$$

### Base Magic Resistance

Base Magic Resistance is a value that never changes throughout a game. It consists of one fixed value set for each unit individually. The base magic resistance of a unit can be a negative number. All 123 heroes currently have the same Base Magic Resistance value: 25%.

### Base Magic Resistance Negation

Some abilities negate or ignores a unit's base magic resistance by removing a percentage of the base magic resistance on the target. This magic resistance negation only affects the base magic resistance value of units, other sources of magic resistance modifiers are not affected.

### Total Magic Damage

Calculating the total magic damage of an ability involves using a few multipliers, Magic Resistance Multiplier, Actual Base Magic Resistance, Magic Damage Barrier and Magic Damage Negation Sources (if any). The total or actual magic damage dealt is equal to how much health a target immediately loses as a direct result.

Actual magic damage dealt after magic resistance is defined as

$$MagicDamageDealt = (MagicalDamage  - Barrier - Negations) * (1 - MagicResistance)$$

### Effective HP

The total amount of magic damage a unit can take due to magic resistance is known as effective HP (or EHP). Despite each source of magic resistance increasing the magic resistance value less the higher it is and making them seem less effective, each sources increases the unit's effective HP against magic damage by their base value.

This definition of EHP assumes all damage taken is magic damage. EHP against physical damage (such as basic hero attacks etc.) is modified by armor.

Effective HP against magical damage is defined as

$$Effective HP = \frac{Current HP}{1 - MagicResistance}$$

## Pure Damage

Pure Damage is a damage type that is not reduced by Armor or Magic Resistance. It is also not amplified by magical damage amplification abilities, fully ignores Armor and Damage Block. It does not affect invulnerable units.

There are a few sources that can manipulate Pure Damage, with Dispersion being one of them. This mechanic is called damage reduction. Pure damage affects units with Spell Immunity (because spell immunity does not block damage), but that does not mean that an ability that deals pure damage is able to target spell immune units.
