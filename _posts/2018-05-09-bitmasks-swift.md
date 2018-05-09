---
layout: single
title: "Bitmasks in Swift"
date: 2018-05-09
categories: swift
---

How to represent options that are not exclusive ? Easy, use a [bitmask](https://en.wikipedia.org/wiki/Mask_(computing)) !  

## `NS_OPTIONS` ##

Objective-C has the `NS_OPTIONS` macro to declare an options set :

{% highlight objc %}
typedef NS_OPTIONS(NSInteger, MyOptions) {
   MyOptions1 = 1 << 0,
   MyOptions2 = 1 << 1,
};
{% endhighlight %}
	
But this is error prone. Using it require to do bitwise operations and there is no type checking.
	
{% highlight objc %}
MyOptions options = MyOptions1 | MyOptions2;
if (options & MyOptions1) {
    // OK 
}

typedef NS_OPTIONS(NSInteger, MyOtherOptions) {
    UnrelatedOption = 1 << 0,
};

if (options & UnrelatedOption) {
    // Probably wrong
}

if (options == 3) {
    // There is some code smell here
}
{% endhighlight %}

## `OptionSet` ##

The Swift standard library has the protocol [OptionsSet](https://developer.apple.com/documentation/swift/optionset)

For example it can be used to represent the borders of a rectangle : 

{% highlight swift %}
struct Borders : OptionSet {
    // When creating an option set, include a `rawValue` property in your type declaration.
    let rawValue: Int
    
    // The static members are unique, individual options
    static let top = Borders(rawValue: 1 << 0)
    static let bottom = Borders(rawValue: 1 << 1)
    static let left = Borders(rawValue: 1 << 2)
    static let right = Borders(rawValue: 1 << 3)
    
    // Declare additional preconfigured option set values as static properties initialized with an array literal containing other option values.
    static let sides:Borders = [.left, .right]
    static let all:Borders = [.top, .bottom, .left, .right]
}
{% endhighlight %}

We get all the [`Set` API](https://developer.apple.com/documentation/swift/set) for free :

{% highlight swift %}
// Create an empty set or borders, add values later
var foo:Borders = []
foo.insert([.top, .bottom])
    
// Check if the set contains a specific value (aka. "Bitwise AND")
foo.contains(.bottom) // ⇢ true
    
// Check if the set contains **exactly** some specific values
foo == .top // ⇢ false
foo == [.bottom, .top] // ⇢ true

// Filter the set
foo.intersection(.sides).isEmpty // ⇢ true 

// Also available : union(_:), subtracting(_:), filter(_:), etc...

// Create another _unrelated_ option set
let bar:CardinalDirection = .northEast

// Type checking is enforced
foo == bar // Compilation Error

{% endhighlight %}


## Objective-C compatibility

Unfortunately, `OptionSet` is swift only and can not be bridged to Objective-C.

## Debug

To ease debugging we can also implement `CustomStringConvertible` so the string printed is not the raw value.

{% highlight swift %}
extension Borders : CustomStringConvertible {
    var description : String {
        var shift = 0
        var result = ""
        var value = self.rawValue
        
        while value != 0 {
            if value & 0b1 > 0 {
                result += ["Top ", "Bottom ", "Left ", "Right "][shift]
            }
            shift += 1
            value >>= 1
        }
        return result
    }
}
{% endhighlight %}

    
    