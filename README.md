# Emoji Keywords

Keyword (`:heart:`) to code (`U+2764`) mapping data.

Currently only for [Twemoji.js](https://github.com/twitter/twemoji) (V2) which supports 1661 emojis.

Available as [YAML](/twemoji/v2/keywords.yaml), and [JSON](/twemoji/v2/keywords.json) format.

## How I get the keyword

Please see [this wiki](https://github.com/jollygoodcode/emoji-keywords/wiki/How-I-get-the-keyword).

## Guide to human to use Emoji keywords

## ZWJ Zero Width Joiner

## Emoji with Color

Unicode 8.0 adds [diversity support](http://unicode.org/reports/tr51/#Diversity) to Emojis!

What does this mean is some emojis have a base color plus 5 modifiers.

For example, boy emoji, The base is `:boy`. You can add skin tone modifiers (U+1F3FB ~ U+1F3FF):

```
:boy:
:boy::skin-tone-2:
:boy::skin-tone-3:
:boy::skin-tone-4:
:boy::skin-tone-5:
:boy::skin-tone-6:
```

## Slack keywords limitations

### Some emojis cannot type to Slack but emoji-keywords has them :smirk:

from `:a:` to `:z:` (except `:a:`, `:b:`, `:m:`, `:o:`, `:v:`, `:x:`)

### Some keywords differ from Slack

Because they're impossible to be use and remember as keyword.

| Slack     | emoji-keywords | Unicode |
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
