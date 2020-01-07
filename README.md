## Lesson 1 - SimpleDateFormat is never that simple - not on Android, at least.

Ah, good old days when formatting dates was easy...said _no one_ **_ever_**.

Dealing with dates, is always hard. Especially when we're talking about the limited and outdated JVM implementation of Android. Well, looking at the bright side of it, there's always something new to be learned.

As such, I can think of no better topic to be the first daily lesson than this:

> You don't really want `YYYY`, you want `yyyy`. There *is* a difference.

Let's explain it a little better: `SimpleDateFormat` is the standard class we use to parse dates from a string of a given format into a proper EPOCH date object that we all know and love.

In order for the it's magic to work, SimpleDateFormat requires only the string template, annotatted with a preset collection of letters, as per the [documentation](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html).

But there are two different ways of parsing the year, with _capital Y_ and _lower case Y_, and most of the time, it doesn't make any difference (so it's really hard to catch in a unit test), but in certain conditions, it will misinterpret the year and make a fool out of you.

As per the documentation, you should use `Y` to parse the string as a year counted in weeks, and `y` for regular gregorian calendar year. So what you really want is `y`.

The date when this bug appeared for me was in December 30th of 2019, a Monday, and the date that starts the week 1 of 2020. My parser, using `YYYY` parsed that date as `30-12-2020`. I would not believe it if the user hadn't taken a screenshot. I was intrigued to say the least. Gladly, there was [a nice video on Youtube explaining the issue](https://www.youtube.com/watch?v=D3jxx8Yyw1c), which saved me from hours of frustration. Now, since I know I'll forget it in the future, I'm documenting it here and hopefully saving my future self from hours of frustration. And, if you're reading this, maybe I'm saving *you* a few hours of frustration. You're welcome :).

And here's the snippet to format the date properly:

``` kotlin
return try {
    SimpleDateFormat("dd/MM/yyyy' at 'HH:mm", Locale.getDefault()).format(Date(date))
} catch (e: Exception) {
    null
}
```
