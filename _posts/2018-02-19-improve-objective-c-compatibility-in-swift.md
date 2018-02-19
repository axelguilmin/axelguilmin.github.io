---
layout: single
title: "Improve Objective-C compatibility in Swift"
date: 2018-02-19
categories: iOS Objective-C Swift
---

`UInt?` `Int?` or `Double?` variables are not exposed in Objective-C beacasue they have no direct equivalent.
It is, however, possible to "wrap" them in a NSNumber like so :

#### Read-Only ####
{% highlight swift %}

class Foo {
    var bar:UInt?
}

// MARK: Objective-C Support
extension Foo {
    /// bar is `UInt?` in Swift and `(NSNumber * _Nullable)` in Objective-C
    @objc(bar)
    var z_objc_bar:NSNumber? {
        return bar as NSNumber?
    }
}
{% endhighlight %}

#### Read and Write ####
You can also add a setter, but this introduce a risk of confusion since a decimal can be passed (it would be converted as necessary though).

{% highlight swift %}

class Foo {
    var bar:UInt?
}

// MARK: Objective-C Support
extension Foo {
    /// bar is `UInt?` in Swift and `(NSNumber * _Nullable)` in Objective-C
    @objc(bar)
    var z_objc_bar:NSNumber? {
    	get {
            return bar as NSNumber?
    	}
        set(value) {
            bar = value?.uintValue ?? nil
    	}
    }
}
{% endhighlight %}