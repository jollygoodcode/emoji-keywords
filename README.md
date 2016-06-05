# Emoji Keywords

Keyword (`:heart:`) to code (`U+2764`) mapping data.

Currently only for [Twemoji.js](https://github.com/twitter/twemoji) (V2) which supports 1661 emojis.

Available as [YAML](/twemoji/v2/keywords.yaml), and [JSON](/twemoji/v2/keywords.json) format.

## How I get the keyword

Please see [this wiki](https://github.com/jollygoodcode/emoji-keywords/wiki/How-I-get-the-keyword).

## Guide to human to use Emoji keywords

## ZWJ Zero Width Joiner

Since Unicode 6.0, there is a thing called [Emoji ZWJ Sequences](http://www.unicode.org/reports/tr51/#Emoji_ZWJ_Sequences).

ZWJ stands for ZERO WIDTH JOINER (U+200D).

You can use ZWJ to compose emojis from single emojis!

For example, a couple with heart emoji:

<img width="43" alt="U+1F468 U+200D U+2764 U+FE0F U+200D U+1F468" src="https://cloud.githubusercontent.com/assets/1000669/15807407/4d9bee3e-2b90-11e6-846c-557d138f7501.png">

Can be composed by man (U+1F468), heart (U+2764), man (U+1F468):

<img width="116" alt="screenshot 2016-06-06 02 42 02" src="https://cloud.githubusercontent.com/assets/1000669/15807406/4d75ad0a-2b90-11e6-8be7-9fb1754fb632.png">

Slack use the `-` to represent ZWJ:

```
:man-heart-man:
```

I followed Slack's convention in Emoji Keywords.

See full [Emoji ZWJ Sequences Catalog](http://www.unicode.org/emoji/charts/emoji-zwj-sequences.html).

## Emoji with Color

Unicode 8.0 adds [diversity support](http://unicode.org/reports/tr51/#Diversity) to Emojis!

What does this mean is some emojis have a base color plus 5 modifiers.

For example, boy emoji, The base is `:boy:`. You can add skin tone modifiers (U+1F3FB ~ U+1F3FF):

```
:boy:
:boy::skin-tone-2:
:boy::skin-tone-3:
:boy::skin-tone-4:
:boy::skin-tone-5:
:boy::skin-tone-6:
```

## Differences from Emoji Keywords and Slack keywords

### [Regional Indicator Symbol letters (`1F1E6..1F1FF`)](http://www.unicode.org/Public/emoji/3.0//emoji-data.txt)

Emoji Keywords use `:a:` to `:z:` for regional indicator symbol letters for consistency.

Slack uses `:a:`, `:b:`, `:m:`, `:o:`, `:o2`, `:v:`, `:x:` differently:

| Slack   | Emoji Keywords   | [Unicode Name][full-emoji-data]         | UNICODE |
|-------- | ---------------- | --------------------------------------- | ------- |
| `:a:`   | `:a_negative:`   | NEGATIVE SQUARED LATIN CAPITAL LETTER A | U+1F170 |
| `:b:`   | `:b_negative:`   | NEGATIVE SQUARED LATIN CAPITAL LETTER B | U+1F171 |
| `:m:`   | `:m_circled:`    | CIRCLED LATIN CAPITAL LETTER M          | U+24C2  |
| `:o:`   | `:large_circle:` | HEAVY LARGE CIRCLE                      | U+2B55  |
| `:o2:`  | `:o_negative:`   | NEGATIVE SQUARED LATIN CAPITAL LETTER O | U+1F17E |
| `:v:`   | `:victory_hand:` | VICTORY HAND                            | U+270C  |
| `:x:`   | `:cross_mark:`   | CROSS MARK                              | U+274C  |

[full-emoji-data]: http://unicode.org/emoji/charts/full-emoji-list.html

Slack does not have `:a:`..`:z:` excpet above table mentioned.

### Some keywords differ from Slack

Because they're impossible to remember as keyword.

| Slack     | Emoji Keywords | Unicode |
| --------- | -------------- | ------- |
| `:u5272:` | `:divide:`     | U+1F239 |
| `:u5408:` | `:together:`   | U+1F234 |
| `:u55b6:` | `:operating:`  | U+1F23A |
| `:u6307:` | `:finger:`     | U+1F22F |
| `:u6708:` | `:moon:`       | U+1F237 |
| `:u6709:` | `:exist:`      | U+1F236 |
| `:u6e80:` | `:fullness:`   | U+1F235 |
| `:u7121:` | `:negation:`   | U+1F21A |
| `:u7533:` | `:apply:`      | U+1F238 |
| `:u7981:` | `:prohibit:`   | U+1F232 |
| `:u7a7a:` | `:empty:`      | U+1F233 |

## Credits

- [Twemoji](https://github.com/twitter/twemoji)
- [Slack](https://slack.com)

## Maintained by Jolly Good Code

[![Jolly Good Code](https://cloud.githubusercontent.com/assets/1000669/9362336/72f9c406-46d2-11e5-94de-5060e83fcf83.jpg)](http://www.jollygoodcode.com)

We specialise in rapid development of high quality MVPs. [Hire us](http://www.jollygoodcode.com/#get-in-touch) to turn your product idea into reality.
