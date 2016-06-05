# Emoji Keywords

Keyword (`:heart:`) to code (`U+2764`) mapping data.

Currently only for [Twemoji.js](https://github.com/twitter/twemoji) (V2) which supports 1661 emojis.

Available as [YAML](/twemoji/v2/keywords.yaml), and [JSON](/twemoji/v2/keywords.json) format.

## How I get the data

I grab the [Twemoji.js V2 preview](http://twitter.github.io/twemoji/2/test/preview.html) html.

And parse all unicode out of `img` tags:

```html
<img draggable="false" class="emoji" alt="🀄" src="https://twemoji.maxcdn.com/2/72x72/1f004.png">
```

Post all the unicodes to a Slack channel, get the html from the browser.

Parse the keywords out of the html got from Slack.

Map these keywords to the unicode. Voila!

But I have made some changes (fixes?) to keywords I got from Slack.

## Maintained by Jolly Good Code

[![Jolly Good Code](https://cloud.githubusercontent.com/assets/1000669/9362336/72f9c406-46d2-11e5-94de-5060e83fcf83.jpg)](http://www.jollygoodcode.com)

We specialise in rapid development of high quality MVPs. [Hire us](http://www.jollygoodcode.com/#get-in-touch) to turn your product idea into reality.
