A naive method to solve Wordle is :
1. get a list of 5-letter english words,
2. calculate the frequency of each letter,
3. test the word which uses the 5 most frequent letters,
4. remove from the list of words those which do not match the game clues,
5. return to step 2. until the game ends.

It's a kind of "clever" brute-force approach. I guess it works most of the time.

But I was not satisfied.

Let's have a look at view-source:https://www.nytimes.com/games/wordle/index.html

It is clear that the interesting stuff must be at the end :
```html
  <body>
    <script>
      (function () {
        // Defining the hash before the main bundle allows the bundle access window.hash
        window.wordle = window.wordle || {};
        window.wordle.hash = '9622bc55';
      })();
    </script>
    <script data-class='loading' data-src="main.9622bc55.js"></script>
    <script
      defer
      type="text/java-script"
      data-class='loading' data-src="https://www.nytimes.com/games-assets/gdpr/cookie-notice-v2.1.2.min.js"
    ></script>
    <game-app></game-app>
  </body>
```

Let's save main.9622bc55.js and open it in a text editor to get a more readable view.
For example, in Notepad++ with plugin "XML tools" use "Pretty print".

Scrolling down the formatted output, one can find a list of words in `var ko=["cigar","rebut", ... ` (12973 word long).

Searching for the variable `ko` in the java-script code, it is used here :
```javascript
function Oo(e){
return t=n%ko.length,ko[t]
}
```
called by
```javascript
e.solution=Oo(e.today),e.dayOffset=Io(e.today)
```
where
```javascript
function Io(e){
return Co(To,e)
}
```
and
```javascript
function Co(e,t){
var n=new Date(e),a=new Date(t).setHours(0,0,0,0)-n.setHours(0,0,0,0);
return Math.round(a/864e5) // 864e5 means 864 * 10^5 = 86400 * 1000 = the number of ms in 1 day.
}
```
From https://developer.mozilla.org/en-US/docs/Web/java-script/Reference/Global_Objects/Date :
> Date objects contain a Number that represents milliseconds since 1 January 1970 UTC

So, in function `Co` above, `a` is equal to number of days * 86400 * 1000 between `Date e` and `Date t`, thus function `Co` returns the number of days between `Date e` and `Date t`.

Looking for variable `To` yields :
`var To=new Date(2021,5,19,0,0,0,0);` equivalent to May 19th 2021 0h0m0s.

Putting all things together :
* array ko[] contains the list of solutions ;
* `var To=new Date(2021,5,19,0,0,0,0)` is the "starting" date
```javascript
function Co(e,t){
var n=new Date(e),a=new Date(t).setHours(0,0,0,0)-n.setHours(0,0,0,0);
return Math.round(a/864e5)
}

function Oo(e){
return t=n%ko.length,ko[t]
}

var t,n=Io(e); // n = Io(e) = Co(To,e)

function Io(e){
return Co(To,e)
}
```

* Function `Oo` computes `t` = the number of elapsed days since May 19th 2021 modulo the size of the array `ko[]` holding the words and returns `ko[t]`.

OK, I have seen enough ugly java-script code, my eyes are shedding tears of blood :see_no_evil:
