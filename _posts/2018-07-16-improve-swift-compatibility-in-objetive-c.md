---
layout: single
title: "Improve Swift compatibility in Objective-C"
date: 2018-07-16
categories: iOS Swift Objective-C
---

There are several scenarios where you want to expose Objective-C APIs to Swift; like slowly introducting Swift in a project, or creating an iOS framework*.

The heavy lifting is done automatically by the compiler, you just need to include the Objective-C headers you wish to expose to Swift in the famous `{project}-Bridging-Header.h` and magic happens. (See [Mix and Match: Objective-C and Swift](https://medium.com/@JoyceMatos/mix-and-match-objective-c-and-swift-c6bceb7f81f3) by Joyce Matos)

However, the two languages have different conventions and features. The bridge created by default might not be the best out of the box.

We will go through an example and introduce some tips that help improving Swift compatibility when writing Objective-C :

- Renaming Objective-C APIs for Swift
- Class properties
- Collections lightweight generics
- Nullability

_* If you are considering writing an iOS framework in Swift, watch the talk [Binary Frameworks in Swift](https://www.dotconferences.com/2018/01/peter-steinberger-binary-frameworks-in-swift) Peter Steinberger gave during dotSwift 2018._

## The example ##

As an example we will use this Objective-C header file :

{% highlight objc %}
typedef NS_ENUM(NSUInteger, MYSingletonState) {
    MYSingletonStateOK,
    MYSingletonStateError
};

@interface MYSingleton : NSObject

+ (MYSingleton *)sharedInstance;
+ (instancetype)new UNAVAILABLE_ATTRIBUTE;
- (instancetype)init UNAVAILABLE_ATTRIBUTE;

- (void)loadSomethingNamed:(NSString *)name;
- (void)loadSomethingNamed:(NSString *)name withOptions:(NSArray *)options;
- (void)loadSomethingNamed:(NSString *)name withOptions:(NSArray *)options completion:(void (^)(MYSingletonState))completion;

@end
{% endhighlight %}

This code is written using the `MY` prefix; it creates an enum called `MYSingletonState` and the interface for the class `MYSingleton`.

`MYSingletonState` has two values, `ok` and `error`.

`MYSingleton` contains boilerplate code to be a singleton, that is the class method `sharedInstance` and the macro `UNAVAILABLE_ATTRIBUTE` to disable the constructors.  

The singleton exposes 3 versions of the same method `loadSomething`. Theses methods load "something" and takes at least one parameter which is a string; an array of options can be passed and a completion handler is called when the loading is done. The completion handler receives `ok` or `error` as the loading succeded or failed.

Using the default Swift bridge, we can use the class like so :

{% highlight swift %}
MYSingleton.sharedInstance().loadSomethingNamed("Foo", withOptions: ["Bar"]) { state in
    if state == MYSingletonState.OK {
        // It worked 
    }
}
{% endhighlight %}

This code works as intented, however it does not fully respect Swift conventions. We would prefer something like : 

{% highlight swift %}
Singleton.shared.loadSomething(name:"Foo", options: ["Bar"]) { state in
    if state == .ok {
        // It worked 
    }
}
{% endhighlight %}

![Let's get Swifty]({{ "/assets/2018-07-16-improve-swift-compatibility-in-objetive-c/get-swifty.jpg" }})

## Renaming Objective-C APIs for Swift ##

The [`NS_SWIFT_NAME`](https://developer.apple.com/documentation/swift/objective_c_and_c_code_customization/renaming_objective_c_apis_for_swift) macro allows to customize API names for Swift.

It is straight forward to use, add it as a prefix (for interfaces) or suffix (for properties, methods, types) and provide the name you want the API to take.

We can use it to get rid of the `MY` prefix, rename `sharedInstance` to `shared` and give `loadSomething` better parameters names :

{% highlight objc %}
typedef NS_ENUM(NSUInteger, MYSingletonState) {
    MYSingletonStateOK NS_SWIFT_NAME(ok),
    MYSingletonStateError
} NS_SWIFT_NAME(SingletonState);

NS_SWIFT_NAME(Singleton)
@interface MYSingleton : NSObject

+ (instancetype)sharedInstance NS_SWIFT_NAME(shared());
// ...
- (void)loadSomethingNamed:(NSString *)name withOptions:(NSArray<NSString *> *)options completion:(void (^)(MYSingletonState))completion NS_SWIFT_NAME(loadSomething(name:options:completion:));
 
@end
{% endhighlight %}

This is already way better, now the call is :

{% highlight swift %}
Singleton.shared().loadSomething(name: "Foo", options: ["Bar"]) { state in
    if state == .ok {
        // It worked
    }
}
{% endhighlight %}

## Class properties ##

We notice that the singleton instance is accessed using a method instead of a parameter, this can be improved thanks to class properties. Class properties were introduced with Xcode 8, as the release note says :

> Objective-C now supports class properties, which interoperate with Swift type properties. They are declared as: @property (class) NSString *someStringProperty;. They are never synthesized. (23891898)
 
Replacing 

{% highlight objc %}
+ (instancetype)sharedInstance NS_SWIFT_NAME(shared()); 
{% endhighlight %}

By
{% highlight objc %}
@property (class, nonatomic, copy, readonly) MYSingleton *sharedInstance NS_SWIFT_NAME(shared);
{% endhighlight %}

Does the trick !

## Collections lightweight generics ##

If we trigger the autocompletion we notice that the type of `options` in Swift is `[Any]!`, its content type can be specified using [lightweight generics](http://www.thomashanning.com/objective-c-lightweight-generics/) like so :

{% highlight objc %}
NSArray<NSString *>*
{% endhighlight %}

Autocompletion in Swift now asks for `[String]!` for the options parameter.
The type of the the parameter will be enforced at compilation time, this is good for both your Objective-C and Swift code !

Note: This only works for `NSArray`, `NSDictionary` and `NSSet`, not their mutable versions.

## Nullability ##

We also notice that autocompletion asks for implicitly unwrapped optionals (aka "!").

This is the default behavior of the compiler because there is no direct equivalent to Optionals in Objective-C.

However annotations are provided for us to refine how the compiler bridges pointers.
Apple explained them in [this article](https://developer.apple.com/swift/blog/?id=25).

__Annotations__

You can annotate objects and blocks pointers with `nullable`, `nonnull` or `null_unspecified`. Either in the property attributes list or immediately after an open parenthesis.

The annoted pointers are then bridged to Optionals (`nullable`), plain (`nonnull`) or Implicitly Unwrapped Optionals (`null_unspecified`).

Things get more complicated if you use double pointers or blocks returning pointers, where you will need to use the underscored forms `_Nullable` and `_Nonnull`.

This [answer](https://stackoverflow.com/a/33682230/1327557) on Stack Overflow gives multiple examples. 

__Audited Regions__

You can mark your code with `NS_ASSUME_NONNULL_BEGIN` and `NS_ASSUME_NONNULL_END` and pointers will be assumed `nonnull`.

> To ease adoption of the new annotations, you can mark certain regions of your Objective-C header files as audited for nullability. Within these regions, any simple pointer type will be assumed to be nonnull.

Note: There is no way to bridge a basic type (like `NSInteger`) as an Optional. Using `NSNumber` instead might be a workaround in some cases but I would not recommend it.

## Wrapping up ##

After applying the tips we learnt, our example code is :

{% highlight objc %}
NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSUInteger, MYSingletonState) {
    MYSingletonStateOK NS_SWIFT_NAME(ok),
    MYSingletonStateError
} NS_SWIFT_NAME(SingletonState);

NS_SWIFT_NAME(Singleton)
@interface MYSingleton : NSObject

@property (class, nonatomic, copy, readonly) MYSingleton *sharedInstance NS_SWIFT_NAME(shared);
+ (instancetype)new UNAVAILABLE_ATTRIBUTE;
- (instancetype)init UNAVAILABLE_ATTRIBUTE;

- (void)loadSomethingNamed:(NSString *)name NS_SWIFT_NAME(loadSomething(name:));
- (void)loadSomethingNamed:(NSString *)name withOptions:(nullable NSArray<NSString *> *)options NS_SWIFT_NAME(loadSomething(name:options:));
- (void)loadSomethingNamed:(NSString *)name withOptions:(nullable NSArray<NSString *> *)options completion:(nullable void (^)(MYSingletonState))completion NS_SWIFT_NAME(loadSomething(name:options:completion:));

@end

NS_ASSUME_NONNULL_END
{% endhighlight %}

It is more verbose than the original but the bridge is swiftier. Calls from Swift are more conventionals:

{% highlight swift %}
Singleton.shared.loadSomething(name: "Foo")
Singleton.shared.loadSomething(name: "Foo", options: ["Bar"])
Singleton.shared.loadSomething(name: "Foo", options: ["Bar"]) { state in
    if state == .ok {
        // It worked
    }
}
{% endhighlight %}

---

In conclusion, we notice that we still can not pass a completion block without providing options. 
We will cover more advanced techniques to fill this gap in a next article.

