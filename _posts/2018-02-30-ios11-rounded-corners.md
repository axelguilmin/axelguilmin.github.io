---
layout: single
title: "iOS11 rounded corners"
date: 2018-01-30
categories: iOS UI
---

iOS 11.0 gifted us with some new APIs for rounded corners.

## maskedCorners

Use [`CALayer.maskedCorners`][apple-masked-corners] to select the corners you want rounded individually.

{% highlight swift %}
// In a UIView
if #available(iOS 11.0, *) {
  layer.cornerRadius = 2.0
  layer.masksToBounds = true
  // Only the bottom corners are rounded
  layer.maskedCorners = [.layerMinXMaxYCorner, .layerMaxXMaxYCorner]
}
{% endhighlight %}

## continuousCorners

This **private API*** allow to draw nicer rounded corners.


<blockquote class="twitter-tweet" data-lang="fr"><p lang="en" dir="ltr">CALayer on iOS 11 has a private &quot;continuousCorners&quot; property, which is what powers many rounded corners in SpringBoard – and likely more! Now I&#39;m jealous. <a href="https://t.co/oPal3NuUiP">pic.twitter.com/oPal3NuUiP</a></p>&mdash; Vlas Voloshin (@argentumko) <a href="https://twitter.com/argentumko/status/955773459463790592?ref_src=twsrc%5Etfw">23 janvier 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

It also works with `maskedCorner`.

\* Using can lead to App Store review rejection, [use with care][gist]

[gist]: https://gist.github.com/efremidze/29e3bf7309727a128928d1a3036a9163
[apple-masked-corners]: https://developer.apple.com/documentation/quartzcore/calayer/2877488-maskedcorners
