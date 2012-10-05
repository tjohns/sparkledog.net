---
layout: post
title: Android compatibility made easy
published: false
---

Backwards compatibility. If you're an application developer these words send chills down your spine. You want to take advantage of the newest features available as your platform evolves, but you don't want to abandon users running older versions.

If you're an Android developer, there's are two [documented options for writing compatibile code][compatibility-blog], published in a blog post from 2009. Here's the short version:

- Option 1 - Reflection: Use `class.getMethod` to get a handle to any potentially incompatible methods. If the class isn't available, a `NoSuchMethodException` can be caught to execute a second code path.
- Option 2 - Wrapper class: Put all of your incompatible code inside a class, then instantiate it inside a _second_ class using `Class.forName()`. If you get an exception during instantiation, you know the class contans unresolved methods.

In both cases, I'm trying to avoid a side effect of Dalvik's [bytecode verifier][verifier]. Any invalid method access would cause a class to be marked as invalid, and would be discarded before it even has a chance to run. This can cascade to make more unreachable methods, eventually causing your entire application to get rejected. Yikes!

Option 1's solution is straightfoward. If a method isn't implemented, you'll get an error if you try to call it. (After all, the method doesn't exist!) Using reflection ensures the verifier never sees a reference to the missing method.

Option 2 is more subtle. All of our potentially invalid method calls are used as-is, inside a dedicated "implementation" class. A wrapper class is used to marshall access to the implementation methods, and a static initializer is used to self-destruct the wrapper class if the implementation class is missing. (There's actually a [bug][bug] in the sample code provided, by the way.)

Oh, and it turns out that Option 2 doesn't work anymore.

Android 2.0 [modified the way the verifier works][verifier-change] so that invalid method access no longer causes classes to be discarded. Instead, the access is just replaced with a `throw` opcode. Your app will crash when the invalid method access occurs, but it won't be because of a missing class. Which means the wrapper class never self-destructs.

There's a [complicated way to get around this][compatibility-blog-2] that takes advantage of the fact that the verifier is lazy, and only runs when a class is first accessed. Architecturally, it's quite elegant.

But it turns out almost nobody uses anything less than Android 2.2 anymore[^1], which means I can actually use this new behavior to my advantage to make something that's not just elegant, but _elegantly simple_: 

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

Yup, that's it. Just a simple *if-statement*.

Since the verifier now converts invalid method calls into `throw`s, I can jump over the offending lines. (`SDK_INT` was introduced in Android 1.6 and Build.VERSION_CODES values are decoded at compile time, so neither of those will cause any prolems either.)

So relax, backwards compatibility doesn't need to be scary.

[^1]: Check the [Platform Versions dashboard][dashboard] if you don't believe me.

[compatibility-blog]: http://android-developers.blogspot.com/2009/04/backward-compatibility-for-android.html
[verifier]: http://htmlpreview.github.com/?https://github.com/android/platform_dalvik/blob/master/docs/verifier.html
[bug]: http://code.google.com/p/android/issues/detail?id=13100
[verifier-change]: https://github.com/android/platform_dalvik/commit/0df441364d2721985d4e64a2ab398c4407c1ff55
[comatibility-blog-2]: http://android-developers.blogspot.com/2010/07/how-to-have-your-cupcake-and-eat-it-too.html
[dashboard]: http://android-developers.blogspot.com/2010/07/how-to-have-your-cupcake-and-eat-it-too.html
