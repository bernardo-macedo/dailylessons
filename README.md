## Lesson 1 - SimpleDateFormat is never that simple - not on Android, at least.

**Ah, good old days when formatting dates was easy … said no one ever.**

Dealing with dates is always hard, especially when we’re talking about the limited and outdated JVM implementation in Android. Well, looking at the bright side of it, there’s always something new to be learned.

As such, I can think of no better topic to be the first daily lesson than this:

> You don’t really want `*YYYY*`, you want `*yyyy*`. There is a difference.

Let’s explain it a little better: `SimpleDateFormat` is the standard class we use to parse dates from a string of a given format into a proper EPOCH date object that we all know and love.

For its magic to work, SimpleDateFormat requires only the string template, annotated with a preset collection of letters, as per the [documentation](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html).

But there are two different ways of parsing the year, with *capital Y* and *lower case Y*, and most of the time, it doesn’t make any difference (so it’s really hard to catch in a unit test). Still, in certain conditions, it is misinterpreting the year and make a fool out of you.

As per the documentation, you should use `Y` to parse the string as a year counted in weeks, and `y` for the regular Gregorian calendar year. So what you really want is `y`.

The date when this bug appeared for me was on December 30th of 2019, a Monday in the standard Gregorian calendar, but that Monday also that starts week 1 of 2020!

My parser, using `YYYY` parsed that date as `30-12-2020`. I would not believe it if the user hadn't taken a screenshot. I was intrigued, to say the least. Gladly, there was [a nice video on Youtube explaining the issue](https://www.youtube.com/watch?v=D3jxx8Yyw1c). 

Now, since I know I'll forget it in the future, I'm documenting it here and hopefully saving my future self from hours of frustration. And, if you're reading this, maybe I'm saving *you* a few hours of frustration. You're welcome, :).

And here’s the snippet to format the date properly:

```kotlin
return try {
    SimpleDateFormat("dd/MM/yyyy' at 'HH:mm", Locale.getDefault()).format(Date(date))
} catch (e: ParseException) {
    null // Do proper error handling
}
```

## Lesson 2 - SimpleDateFormat makes a comeback!

Let's make this lesson as a bonus on SimpleDateFormat (and hopefully forget it for a long time!)

> SimpleDateFormat is *lenient* by default. It may not throw an exception even if the template does not match completely the preset pattern.

Let's say we have this code:

```kotlin
val format = SimpleDateFormat("MM/dd/yyyy") // expected input: "02/20/2020"
try {
    val date = format.parse("2020/02/20") // wrong input
    println(date)        
} catch (e: ParseException) {
    println("Exception!")
}
```

What would you expect as output? `Exception!`? Well, me too. But we'd be wrong. This code prints a completely wrong date: `Tue Apr 02 00:00:00 UTC 188`.

This example is obviously wrong and even without exceptions, it's easy to catch with the most basic testing. However, depending on the combination of the expected template and given input, the error may not be as visible, and thus, harder to catch (think milliseconds).

By being lenient, SimpleDateFormat allows our code to get away with some inconsistencies without us realizing it. But that's not what we really want, is it? We want to always show consistent information (and **fail properly** otherwise!).

If we tweak the code above just setting the leniency to false, it makes the date parsing much more strict and consistent with what we expected initially:

```kotlin
val format = SimpleDateFormat("MM/dd/yyyy") 
format.isLenient = false // strict parsing
try {
    val date = format.parse("2020/02/20") // wrong input
    println(date)        
} catch (e: ParseException) {
    println("Exception!")
}
// outputs: Exception!
```

The leniency of SimpleDateFormat exists because it relies on the `Calendar` implementation, which defaults its leniency to `true`. You can find more information in the [official documentation](https://docs.oracle.com/javase/7/docs/api/java/util/Calendar.html).