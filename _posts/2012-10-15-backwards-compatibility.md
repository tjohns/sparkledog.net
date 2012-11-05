---
layout: post
title: Android compatibility made easy
summary: Backwards compatibility. If you’re an application developer these words can send chills down your spine. You want to take advantage of the newest features available as your platform evolves, but you don’t want to abandon users running older OS versions. At best, this means extra work; at worst, maintaining entirely different codebases. If you’re an Android developer, here’s how to keep that extra work to a minimum.
published: true
---

Backwards compatibility. If you're an application developer these words can send chills down your spine. You want to take advantage of the newest features available as your platform evolves, but you don't want to abandon users running older OS versions. At best, this means extra work; at worst, maintaining entirely different codebases. If you're an Android developer, here's how to keep that extra work to a minimum.

There are two [documented options for writing compatibile code][compatibility-blog], published on the Android Developers blog from 2009. Here's the short version:

- **Option 1 - Reflection:** Use `class.getMethod` to get a handle to any potentially incompatible methods. If the class isn't available, a `NoSuchMethodException` can be caught to execute a second code path.
- **Option 2 - Wrapper class:** Put all of your incompatible code inside one class, then reference it inside a _second_ class. This keeps potentially incompatible method calls outside of your core application code.[^1]

In both cases, I'm trying to avoid a side effect of Dalvik's [bytecode verifier][verifier]. Any invalid method access would cause a class to be marked as invalid by the classloader, and would be discarded before it even has a chance to run. If you didn't take precautions — or made a mistake — your core application code could get discarded. Yikes!

To make things simpler Android 2.0 [modified the way the verifier works][verifier-change] so that invalid method access no longer causes classes to be discarded. Instead, the access is just replaced with a `throw` opcode. This would still cause a crash when the method access occurs, only it would be catchable.

There's a [complicated way to get around this][compatibility-blog-2] that takes advantage of the fact that the verifier is lazy, and only runs when a class is first accessed. Architecturally, it's quite elegant.

But it turns out almost nobody uses anything less than Android 2.2 anymore[^2], which means I can actually use this new behavior to my advantage to make something that's not just elegant, but _elegantly simple_: 

{% highlight java %}
import android.util.Log;
import android.os.Build;

private static final String TAG = "MyApp";
private static final String errorMsg = "Insert scary error message here!";

public void doSomething() {
  ...
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.FROYO) {
    Log.wtf(TAG, errorMsg);
    // The Log.wtf() method was introducted in API 8
  } else {
    Log.e(TAG, errorMsg);
    // The Log.e() method has existed since API 1
  }
}
{% endhighlight %}

Yup, that's it. Just a simple *if-statement*.

Since the verifier now converts invalid method calls into exceptions, I can jump over the offending lines. (`SDK_INT` was introduced in Android 1.6 and `Build.VERSION_CODES` values are decoded at compile time, so neither of those will cause any prolems either.)

So relax, backwards compatibility doesn't need to be scary.

[^1]: It actually turns out that wrapper classes don't work anymore, and there's a [bug][bug] in the  provided sample code. I don't recommend using them.
[^2]: Check the [Platform Versions dashboard][dashboard] if you don't believe me.

[compatibility-blog]: http://android-developers.blogspot.com/2009/04/backward-compatibility-for-android.html
[verifier]: http://htmlpreview.github.com/?https://github.com/android/platform_dalvik/blob/master/docs/verifier.html
[bug]: http://code.google.com/p/android/issues/detail?id=13100
[verifier-change]: https://github.com/android/platform_dalvik/commit/0df441364d2721985d4e64a2ab398c4407c1ff55
[compatibility-blog-2]: http://android-developers.blogspot.com/2010/07/how-to-have-your-cupcake-and-eat-it-too.html
[dashboard]: http://android-developers.blogspot.com/2010/07/how-to-have-your-cupcake-and-eat-it-too.html