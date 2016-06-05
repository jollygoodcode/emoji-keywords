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

How to parse all the unicode? It is too simple that I don't even write any code to do it.

I just multi-select `<img draggable="false" class="emoji" alt=`, remove all of them.

Multi-select ` src="https://twemoji.maxcdn.com/2/`, move to the beginning, delete to the end of line.

result in this [twemoji-js-v2-unicodes](https://gist.github.com/JuanitoFatas/298e6ad5ca8057dfa12ab989ea637c0a) file.

Then Post all the unicodes to a Slack channel:

```ruby
require "http"

class Slack
  def self.post(text)
    payload = {
      channel: "#foo", # <--- channel you want to post
      text: text,
      username: "@_@",
      icon_emoji: ":computer:",
    }.each { |_, v| v.freeze }.freeze

    HTTP.post("https://hooks.slack.com/services/123456789/123456789/12345678901234567", json: payload)
  end
end

# frozen_string_literal: true

require "logger"
require_relative "./slack"

logger = Logger.new("sent.log")

IO.readlines("twemoji-js-v2-unicodes").each_with_index do |line, index|
  cleaned = line.chomp.gsub(?", "")
  logger.info %(Index: #{index}: Post "#{cleaned}" to Slack.)
  Slack.post(cleaned); sleep 0.3
end
```

Then I copy the html from the browser developer tool.

Delete something I am not interested, save to a file, the file structure basically look like this:

```html
<ts-message>
  ...

  <div class="message_content ">
    ...

    <span class="message_body">
      <span class="emoji-outer emoji-sizer emoji-only">
        <span class="emoji-inner" style="background: url(https://a.slack-edge.com/e4cee/img/emoji_2016_02_06/sheet_apple_64_indexed_256colors.png);background-position:10% 72.5%;background-size:4100%" title="mahjong">
          :mahjong:
        </span>
      </span>
    </span>

    ...
  </div>

  ...
</ts-message>

<ts-message>
  ...
</ts-message>

...
```

Then I start to find patterns of them.

There are 3 types of emoji format:

- emoji + skin color ([Diversity](http://unicode.org/reports/tr51/#Diversity)) keyowrds

  ```slim
  span.message_body
    span.emoji-outer.emoji-sizer.emoji-only
      span.emoji-inner title="snowboarder" <--
        :snowboarder: <--
    span.emoji-outer.emoji-sizer
      span.emoji-inner title="skin-tone-2" <--
        :skin-tone-2: <--
  ```

- emoji keyword only

  ```slim
  span.message_body
    span.emoji-outer.emoji-sizer.emoji-only
      span.emoji-inner title="dart" <--
        :dart: <--
  ```

- unicode

  ```slim
  span.message_body
    🇧 <--
  ```


Then I write a `ExtractEmoji` class to extract these 3 kinds of emoji keywords:

```ruby
# frozen_string_literal: true

class ExtractEmoji
  def initialize(html)
    @html = html
  end

  def call
    titles = html.scan %r(title="(?<title>[^"]+)")
    emojis = html.scan %r(>(?<emoji>:[^:]+:)</span>)
    text   = html.scan %r(span class="message_body">(?<unicode>.+)</span>)

    if titles.empty? && emojis.empty? # unicode
      Hash("unicode" => text.join)
    else
      Hash(titles.join(" ") => emojis.join)
    end
  end

  private

    attr_reader :html
end

# Specs
require_relative "./extract_emoji"

RSpec.describe ExtractEmoji do
  describe "#call" do
    def result
      ExtractEmoji.new(html).call
    end

    context "emoji with skin tone" do
      let(:html) do
        <<~HTML
          <span class="message_body"><span class="emoji-outer emoji-sizer emoji-only"><span class="emoji-inner" title="snowboarder">:snowboarder:</span></span><span class="emoji-outer emoji-sizer"><span class="emoji-inner" title="skin-tone-6">:skin-tone-6:</span></span></span>
        HTML
      end

      it "returns" do
        expect(result).to eq Hash(
          "snowboarder skin-tone-6" => ":snowboarder::skin-tone-6:")
      end
    end

    context "emoji only" do
      let(:html) do
        <<~HTML
          <span class="message_body"><span class="emoji-outer emoji-sizer emoji-only"><span class="emoji-inner" title="dart">:dart:</span></span></span>
        HTML
      end

      it "returns" do
        expect(result).to eq Hash("dart" => ":dart:")
      end
    end

    context "unicode" do
      let(:html) do
        <<~HTML
          <span class="message_body">©</span>
        HTML
      end

      it "returns" do
        expect(result).to eq Hash("unicode" => "©")
      end
    end
  end
end
```

Use this `ExtractEmoji` and some nokogiri to parse them out:

```ruby
require "nokogiri"
require_relative "./extract_emoji"

raw_data = IO.read "emoji-dump.html"
document = Nokogiri::HTML::Document.parse raw_data
htmls = document.css("span.message_body").map(&:to_html).map do |html|
  html.gsub(/style="[^"]+"\s{1}/, "") # remove style attributes
end

emoji_lists = htmls.map { |html| ExtractEmoji.new(html).call }
```

However, I find some emojis did not have the Emoji color modifiers, e.g.:

```
{ "santa" => ":santa:" },
{ "santa" => ":santa:" },
{ "santa" => ":santa:" },
{ "santa" => ":santa:" },
{ "santa" => ":santa:" },
{ "santa" => ":santa:" },
```

So I write a `AppendSkinColors` to append `skin-tone-{2,3,4,5,6}` to fix those:

```ruby
# frozen_string_literal: true

class AppendSkinTones
  def initialize(emoji_lists)
    @emoji_lists = emoji_lists
  end

  def call
    color_index = 2

    emoji_lists.each_with_index do |current, index|
      if current == emoji_lists[index+1]
        title, emoji = current.to_a.flatten
        emoji_lists[index] = append_skin_tone(title, emoji, color_index)
        color_index += 1
      else
        color_index = 2
      end
    end
  end

  private

    attr_reader :emoji_lists

    def append_skin_tone(title, emoji, color_index)
      {
        "#{title} skin-tone-#{color_index}" => "#{emoji}:skin-tone-#{color_index}:"
      }
    end
end

# Specs
require_relative "./append_skin_tones"

RSpec.describe AppendSkinTones do
  describe "#call" do
    let(:emoji_lists) do
      [
        { "christmas_tree" => ":christmas_tree:" },
        { "santa" => ":santa:" },
        { "santa" => ":santa:" },
        { "santa" => ":santa:" },
        { "santa" => ":santa:" },
        { "santa" => ":santa:" },
        { "santa" => ":santa:" },
        { "fireworks" => ":fireworks:" },
        { "snowboarder skin-tone-2" => ":snowboarder::skin-tone-2:" },
        { "snowboarder skin-tone-3" => ":snowboarder::skin-tone-3:" },
        { "snowboarder skin-tone-4" => ":snowboarder::skin-tone-4:" },
        { "snowboarder skin-tone-5" => ":snowboarder::skin-tone-5:" },
        { "snowboarder skin-tone-6" => ":snowboarder::skin-tone-6:" },
        { "snowboarder" => ":snowboarder:" },
        { "nose" => ":nose:" },
        { "nose" => ":nose:" },
        { "nose" => ":nose:" },
        { "nose" => ":nose:" },
        { "nose" => ":nose:" },
        { "nose" => ":nose:" },
      ]
    end

    def result
      AppendSkinTones.new(emoji_lists).call
    end

    it "works" do
      expect(result).to eq(
        [
          { "christmas_tree" => ":christmas_tree:" },
          { "santa skin-tone-2" => ":santa::skin-tone-2:" },
          { "santa skin-tone-3" => ":santa::skin-tone-3:" },
          { "santa skin-tone-4" => ":santa::skin-tone-4:" },
          { "santa skin-tone-5" => ":santa::skin-tone-5:" },
          { "santa skin-tone-6" => ":santa::skin-tone-6:" },
          { "santa" => ":santa:" },
          { "fireworks" => ":fireworks:" },
          { "snowboarder skin-tone-2" => ":snowboarder::skin-tone-2:" },
          { "snowboarder skin-tone-3" => ":snowboarder::skin-tone-3:" },
          { "snowboarder skin-tone-4" => ":snowboarder::skin-tone-4:" },
          { "snowboarder skin-tone-5" => ":snowboarder::skin-tone-5:" },
          { "snowboarder skin-tone-6" => ":snowboarder::skin-tone-6:" },
          { "snowboarder" => ":snowboarder:" },
          { "nose skin-tone-2" => ":nose::skin-tone-2:" },
          { "nose skin-tone-3" => ":nose::skin-tone-3:" },
          { "nose skin-tone-4" => ":nose::skin-tone-4:" },
          { "nose skin-tone-5" => ":nose::skin-tone-5:" },
          { "nose skin-tone-6" => ":nose::skin-tone-6:" },
          { "nose" => ":nose:" },
        ]
      )
    end
  end
end
```

Finally re-run all these together again:

```ruby
require "nokogiri"
require "json"
require_relative "./extract_emoji"
require_relative "./append_skin_tones"

raw_data = IO.read "emoji-dump.html"

document = Nokogiri::HTML::Document.parse raw_data

htmls = document.css("span.message_body").map(&:to_html).map do |html|
  html.gsub(/style="[^"]+"\s{1}/, "")
end

emoji_lists = htmls.map { |html| ExtractEmoji.new(html).call }

with_skin_colors = AppendSkinTones.new(emoji_lists.dup).call

IO.write "emoji-lists", with_skin_colors.join("\n")
```

Then I take the keywords from `emoji-lists` and merge with `twemoji-js-v2-unicodes` file.

Resulting in this `TwemojiJS::MAP` constant:

```
class TwemojiJS
  MAP = {
    ":mahjong:" => "1f004",
    ":black_joker:" => "1f0cf",
    ":a_negative:" => "1f170",
    ":b_negative:" => "1f171",
    ...
    ":copyright:" => "a9",
    ":registered_sign:" => "ae",
    ":shibuya:" => "e50a",
  }
end
```

Then load this constant in the irb console.

Because Ruby will tell you if any duplication exists.

I manully fix all the duplications.

Then generate the YAML and JSON:

```ruby
require "yaml"; require "json"
IO.write "keywords.yaml", TwemojiJS::MAP.to_yaml
IO.write "keywords.json", TwemojiJS::MAP.to_json
```

Voila!

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
