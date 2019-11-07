# Middlemanで\\nLazy image loading
subtitle
: 2019-11-07

subtitle
: 表参道.rb #52

author
: うなすけ

theme
: unasuke-white

# 自己紹介
- 名前 : うなすけ
- 仕事 : フリーランスのプラグラマー
  - インフラ寄りサーバーサイドエンジニア
  - Ruby, Rails, Kubernetes...

- {::tag name="x-small"}GitHub [@unasuke](https://github.com/unasuke){:/tag}
- {::tag name="x-small"}Mastodon [@unasuke@mstdn.unasuke.com](https://mstdn.unasuke.com/@unasuke){:/tag}
- {::tag name="x-small"}Twitter [@yu\_suke1994](https://twitter.com/yu_suke1994){:/tag}

![](img/icon_raw.jpg){:relative_width="24" align="right" relative_margin_right="-10" relative_margin_top="42"}

# 前回のあらすじ
1. `<img loading="lazy">` が良さげだし使ってみたい
1. でもMarkdownでどうやって……？
1. Redcarpetならrenderingを拡張できるぞ！
1. `redcarpet-render-html_lazy_img` gemができました

<https://slide.rabbit-shocker.org/authors/unasuke/omotesandorb-51/>

# Middlemanに組み込んでみる
```ruby
set :markdown, renderer: Redcarpet::Render::HTMLLazyImg::Lazy
```

手元で↓になってることを確認

```html
<img src="https://example.org/image.png" loading="lazy" />
```

# 悲しいことに

{:.center}
{::tag name="x-large"}Deployしたら画像が全部\\nリンク切れになってしまった{:/tag}

# リンク切れ、なぜ

{:.center}
↓↓元凶はこれだった↓↓

```ruby
activate :asset_hash
```

(assetにdigestをくっつけてcacheされないようにするもの)

# なぜ asset_hash を有効にするとダメ？
**A.** MiddlemanもRedcarpetのRendererを独自に拡張しているから

```ruby
def link(link, title, content)
  if !@local_options[:no_links]
    attributes = { title: title }
    attributes.merge!(@local_options[:link_attributes]) if @local_options[:link_attributes]

    scope.link_to(content, link, attributes)
  else
    link_string = link.dup
    link_string << %("#{title}") if title && !title.empty? && title != alt_text
    "[#{content}](#{link_string})"
  end
end
```
<https://github.com/middleman/middleman/blob/master/middleman-core/lib/middleman-core/renderers/redcarpet.rb>

# ここまでのまとめ
- MiddlemanもRedcarpetを拡張している
- 外部からRendererを挿入すると予期せぬ挙動
  - 特に asset_hash
- それでも `loading="lazy"` したい

# じゃあどうするか
- Middleman拡張をつくるしかないじゃんね
  - ドキュメントがある
  - これまでに2つ作ったことがある、できるはず
    - middleman-somemoji
    - middleman-hatenastar

<https://middlemanapp.com/jp/advanced/custom-extensions/>

# Middleman拡張
- いくつかのコールバックを登録できる
  - after_build がよさそう
  - 生成されたhtml内の `<img>` を書き換える

```ruby
def after_build(builder)
  files = Dir.glob(File.join(app.config[:build_dir], "**", "*.html"))
  files.each do |file|
    contents = File.read(file)
    replaced = contents.gsub(%r[<img], "<img loading=\"#{options[:loading]}\"")
    File.open(file, 'w') do |f|
      f.write replaced
    end
  end
end
```

# できた
<https://github.com/unasuke/middleman-img_loading_attribute>

(本番投入まだだけど多分これでいける)

(この実装だとpreの中の `<img>` も書き換えてしまうね？！)

# まとめ
- `<img loading="lazy">` を使うとき
  - Markdownなら **redcarpet-render-html_lazy_img**
  - Middlemanなら **middleman-img_loading_attribute**
- 今日はこれを覚えて帰ってください
