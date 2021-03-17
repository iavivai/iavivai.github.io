---
layout: post
title: DiscordにRSSフィードを流す
---

DiscordにRSSフィードを流したい時があったので、その時の作業まとめを書く。
ちなみにRSSリーダーは主にFeedlyを使っている。

今回はGoogle アラートのフィードを流したかったので、Google アラートのフィードURLを指定して実装しているが、どのフィードURLでも基本の実装は変わらないと思う。

## 作業内容
Google アラートの配信先がデフォルトだとメールになっているので、メールからRSSフィードに変更する。
DiscordはSlackみたいに公式でRSSの機能があるわけではないので、RSSを取得するところは自分で実装した。

ソースコードはこちら。
```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

require 'rss'
require 'dotenv/load'
require 'discordrb/webhooks'

CLIENT = Discordrb::Webhooks::Client.new(url: ENV['POKOPEA_WEBHOOK_URL'])

urls = [
  ENV['PEANUTSKUN_ENDPOINT'],
  ENV['PONPOKO_ENDPOINT'],
  ENV['POKOPEA_ENDPOINT']
]

items = urls.map do |url|
  RSS::Parser.parse(url).items.map do |item|
    {
      title: item.title.content,
      description: item.content.content,
      url: item.link.href,
      timestamp: item.updated.content
    }
  end
rescue RSS::MissingTagError => e
  puts "#{e.class}: #{e.message}"
end.flatten.compact

if items.any?
  CLIENT.execute do |builder|
    items.each do |item|
      builder.add_embed do |embed|
        embed.title = item[:title]
        embed.description = item[:description]
        embed.url = item[:url]
        embed.timestamp = item[:timestamp]
      end
    end
  end
else
  puts '新着情報はありません'
end
```
[discord_bot/alert_pokopea at master · utakah/discord_bot · GitHub](https://github.com/utakaha/discord_bot/blob/master/bin/alert_pokopea)

流したいフィードURLをループして、新着情報があればその情報をWebhook URLで指定したチャンネルに流すというコードを書いた。
このスクリプトを毎朝10時に実行するようにスケジューラを設定している。

---
今回はDiscordのサーバーに流したくてコードを書いたけど、そもそも個人利用ではFeedlyが最強だと思う。
