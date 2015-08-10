# vr.scss

「vr.scss」は「Vertical Rhythm」をSassで実現するための@mixinです。remをベースに、レスポンシブ対応しています。

[DEMO](http://codepen.io/gaku/pen/ZGjZyJ)

## 変数
最終的な出力結果はremですが、変数はすべてpxで指定します。`$base-font-size`と`$base-spacing`にサイズを指定してください。行の高さとフォントサイズを割った数値が`24/16=1.71429`のように小数点の桁が大きい場合はブラウザでpxに変換される時にズレが生まれやすいので、terkei(https://github.com/terkel)さんが作られた`decimal-round`関数で小数点以下3桁までに指定しています。

```scss
$base-font-size: 14px !default;
$base-spacing: 24px !default;
$base-line-height: decimal-round($base-spacing / $base-font-size, 3, ceil) !default;
```

行の高さやフォントサイズをレスポンシブに変更したい場合は`$shift-vertical-rhythm`をtrueにします。`$shift-`から始まる変数はレスポンシブ用の変数になります。

```scss
$shift-vertical-rhythm: true !default;
$shift-font-size: 16px !default;
$shift-spacing: 28px !default;
$shift-line-height: ($shift-spacing / $shift-font-size) !default;
```

ブレイクポイントの`$vr`はデフォルトで1000px以上になっています。

```scss
$breakpoints: (
    'palm': (min-width: 400px),
    'tab': (min-width: 768px),
    'lap': (min-width: 1000px),
    'desk': (min-width: 1200px),
    'below-palm': (max-width: 399px),
    'below-tab': (max-width: 767px),
    'below-lap': (max-width: 999px),
    'below-desk': (max-width: 1199px),
    'vr': (min-width: 1000px)
) !default;
```

## @mixin

@mixinは3つ用意しています。

1. メディアクエリ用の@mixin
1. `font-size`と`line-height`を返す@mixin
1. `margin-top`と`margin-bottom`を返す@mixin

```scss
@mixin media-query($breakpoint) {
    @if map-has-key($breakpoints, $breakpoint) {
        @media only screen and #{inspect(map-get($breakpoints, $breakpoint))} {
            @content;
        }
    }
    @else {
        @warn "Unfortunately, no value could be retrieved from `#{$breakpoint}`. "
        + "Available breakpoints are: #{map-keys($breakpoints)}.";
    }
}

@mixin vr-font-size($font-size, $font-size-shift: $font-size) {
    font-size: ($font-size / $base-font-size) * 1rem;
    line-height: $base-line-height * (ceil($font-size / $base-spacing) * 1rem);
    @if $shift-vertical-rhythm == true {
        @include media-query(vr) {
            font-size: ($font-size-shift / $shift-font-size) * 1rem;
            line-height: $shift-line-height * (ceil($font-size-shift / $shift-spacing) * 1rem);
        }
    }
}

@mixin vr-spacing($top: 0, $bottom: 1) {
    $top-unit: 1rem;
    $bottom-unit: 1rem;
    @if $top == 0 {
        $top-unit: 0;
    }
    @if $bottom == 0 {
        $bottom-unit: 0;
    }
    margin-top: ($base-line-height * $top-unit) * $top;
    margin-bottom: ($base-line-height * $bottom-unit) * $bottom;
    @if $shift-vertical-rhythm == true {
        @include media-query(vr) {
            @if $top != 0 {
                margin-top: ($shift-line-height * $top-unit) * $top;
            }
            @if $bottom != 0 {
                margin-bottom: ($shift-line-height * $bottom-unit) * $bottom;
            }
        }
    }
}
```

## scss

`html`に対してベースになる`font-size`と`line-height`を指定します。変数の`$shift-vertical-rhythm`にfalseが指定されている場合は@if以降は出力されません。

```scss
html {
    font-size: $base-font-size;
    line-height: $base-line-height;
    @if $shift-vertical-rhythm == true {
        @include media-query(vr) {
            font-size: $shift-font-size;
            line-height: $shift-line-height;
        }
    }
}
```

見出しなどフォントサイズを変更する要素には`vr-font-size()`を指定します。第一引数はベース、第二引数はブレイクポイント以降のフォントサイズになります。`line-height`は`font-size`が`line-height`を超えていないかを`ceil`関数（小数点の切り上げ）によって判定をして、超えていなければ1を、超えていれば2以上を掛けます。

1. font-sizeが15px、line-heightが24pxで引数に24pxを指定した場合は`24/24=1`なので1倍が指定される。
1. font-sizeが15px、line-heightが24pxで引数に25pxを指定した場合は`25/24=1.04167`なので2倍が指定される。
1. font-sizeが15px、line-heightが24pxで引数に48pxを指定した場合は`48/24=2`なので2倍が指定される。
1. font-sizeが15px、line-heightが24pxで引数に49pxを指定した場合は`49/24=2.04167`なので3倍が指定される。

要素に上下のマージンを指定する場合は`vr-spacing()`を指定します。引数の数値（整数）に応じた行数が返されます。

```scss
h1 { @include vr-font-size(26px, 30px); }
h2 { @include vr-font-size(20px, 24px); }
h3 { @include vr-font-size(18px, 22px); }

h1 { @include vr-spacing(2,2); }
h2,
h3 {
    @include vr-spacing(2,1);
}
p,
ul,
ol,
h4,
h5,
h6 {
    @include vr-spacing();
}
```

以下のようにCSSにコンパイルされます。

```css
html {
    font-size: 15px;
    line-height: 1.6;
}
@media only screen and (min-width: 1024px) {
    html {
        font-size: 16px;
        line-height: 1.75;
    }
}

h1 {
    font-size: 1.73333rem;
    line-height: 3.2rem;
}
@media only screen and (min-width: 1024px) {
    h1 {
        font-size: 1.875rem;
        line-height: 3.5rem;
    }
}

h2 {
    font-size: 1.33333rem;
    line-height: 1.6rem;
}
@media only screen and (min-width: 1024px) {
    h2 {
        font-size: 1.5rem;
        line-height: 1.75rem;
    }
}

h3 {
    font-size: 1.2rem;
    line-height: 1.6rem;
}
@media only screen and (min-width: 1024px) {
    h3 {
        font-size: 1.375rem;
        line-height: 1.75rem;
    }
}

h1 {
    margin-top: 3.2rem;
    margin-bottom: 3.2rem;
}
@media only screen and (min-width: 1024px) {
    h1 {
        margin-top: 3.5rem;
        margin-bottom: 3.5rem;
    }
}

h2,
h3 {
    margin-top: 3.2rem;
    margin-bottom: 1.6rem;
}
@media only screen and (min-width: 1024px) {
    h2,
    h3 {
        margin-top: 3.5rem;
        margin-bottom: 1.75rem;
    }
}

p,
ul,
ol,
h4,
h5,
h6 {
    margin-top: 0;
    margin-bottom: 1.6rem;
}
@media only screen and (min-width: 1024px) {
    p,
    ul,
    ol,
    h4,
    h5,
    h6 {
        margin-bottom: 1.75rem;
    }
}

```

## テスト環境
「Vertical Rhythm」を可視化するため以下のHTMLとscssコードを記述します。ピンクとグレーの線がズレていたら、記述ミスか、行の高さが割り切れていない可能性があります。

```html
<div class="grid">
  <div class="line">&nbsp;</div>
  <div class="line">&nbsp;</div>
  <div class="line">&nbsp;</div>
  <div class="line">&nbsp;</div>
  <div class="line">&nbsp;</div>
</div>
```

```scss
.grid {
    position: absolute;
    z-index: -1;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    .line {
        font-size: 1rem;
        line-height: $base-line-height;
        box-shadow: 0 1px 0 #eee;
        @if $shift-vertical-rhythm == true {
            @include media-query(vr) {
                line-height: $shift-line-height;
            }
        }
    }
}
p,
ul,
ol,
h1,
h2,
h3,
h4,
h5,
h6 {
    box-shadow: 0 1px 0 #faa, inset 0 1px 0 #faa;
}
```