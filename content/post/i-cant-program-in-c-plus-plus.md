---
title: "I Can't Program in C++"
date: 2023-11-18T16:42:42-06:00
description: "Learning C++ by creating a BASIC transpiler"
draft: false
toc: false
categories: ["technology", "programming", "C++"]
tags: ["C++", "jazz"]
---

H. Jon Benjamin, known for voicing Sterling Archer, and Bob in the shows Archer, and Bobs burgers released a Jazz album in 2015 titled: _"Well, I should have... Learned How To Play Piano"_. As the title of the album and the second track "I can't play piano" suggests, Jon can't play piano, nor does he really like Jazz. Feel free to listen to some of the music and idea behind the album [here](https://youtu.be/JuKJkghC2u0). It sounds like jazz... _sort of_. Jon is just following the patterns of what a piano in a jazz ensemble _ought_ to sound like, but not playing the any of the right chords.

I've found myself in a similar position a few times when I'm thrown into a new project at work or something of my own design where I'm using a language I'm not familiar with. I've used enough programming languages to know to how to generally code in just about any language just by following patterns left for me... Ok maybe not `APL`, seriously wtf is this language:

```apl
life ← {⊃1 ⍵ ∨.∧ 3 4 = +/ +⌿ ¯1 0 1 ∘.⊖ ¯1 0 1 ⌽¨ ⊂⍵}
```

The code I end up writing however, is something that is always... not quite idiomatic. My first efforts coding in any new language tend to miss some of the _best_ ways to accomplish a task. Sure it's going to work, but I'm probably missing some key feature of the language that would simplify everything for me. It can also take time to get proficient at using a language to the point were you stop needing to look up basic syntax and you can just get working. Some languages like python or ruby are so simple that you can pick up the basics of the language quickly, and before you know it you feel like you're an expert. C++ is not one of those languages, or so I am told. Recently I found myself in a situation where I need to do a bit of C/C++ development professionally, and rather than just jump in to the code base I wanted to see if I could learn a little before I started cranking out code. All of this is just a really long winded way of saying: Fake it 'til you make it, so I'm going to do just that.

## Picking a project 
A blog post I've always gone back to when I want to challenge myself is Austin Henley's ["Challenging projects every programmer should try"](https://austinhenley.com/blog/challengingprojects.html). So today I'm going to make a BASIC compiler in C++. I suspect that when I'm done I probably still won't be able to program in C++, but I'll know enough that it should compile and run. I'm following along with [the tutorial for building a BASIC](https://austinhenley.com/blog/teenytinycompiler1.html) which is written in python, but it should be pretty simple to follow along in C++.



## Strings

I don't know any thing about C++ at all so, first thing is first, what is wrong here:

```c++
    _currentChar = '';
```

Unlike in python `''` is not allowed for an empty character. This needs to be changed to `_currentChar = '\0';`. Interesting.

The next thing I ran into was `myString.size()` gives you the size in bytes of a string. This really threw me off, if you're using a language like ruby for example, `"asdf".size` will give you the length of a string, in fact it's just an alias of the `length` function. The IDE I used just suggested it and I sort of guessed that's what I needed to be using. What I needed to be using was `length()`, which gives you the number of characters.

`str::substr` Does not work the way you'd expect it to. In python you choose the indicies of the beginning and the end of your string. In CPP, the second parameter is the length! I made this mistake more than once and had to retrace my steps as to why I wasn't parsing my strings correctly. Here's an example of some python code and the corresponding c++ code I wrote to instead. 

```python
startPos = self.curPos
while self.peek().isalnum():
    self.nextChar()

# Check if the token is in the list of keywords.
tokText = self.source[startPos : self.curPos + 1] # Get the substring.
keyword = Token.checkIfKeyword(tokText)
```

```c++
int startPosition = _currentPosition;
int tokenLength = 1;
while (std::isalpha(peek()) || std::isdigit(_currentChar)) {
    nextChar();
    tokenLength++;
}

string tokenText = _source.substr(startPosition, tokenLength);
```

because I needed the length, it just became easier to have counters wherever I used substring.


## Enums

enums are a lot easier to work with in python, period. In python you can just check if a string is a member of the enum class. Not so in C++, there is no built in way of determining if a string is a valid enum of some type. You need to map it all out yourself. That means creating methods like:



```c++
bool isKeyword(const std::string& str) {
	static const std::unordered_set<std::string> keywordSet = {
		"LABEL", "GOTO", "PRINT", "INPUT", "LET", "IF",
		"THEN", "ENDIF", "WHILE", "REPEAT", "ENDWHILE"
	};

	return keywordSet.find(str) != keywordSet.end();
}
```

or

```c++
string tokenTypeToString(TokenType type) {
    switch (type) {
        case INVALIDTOKEN: return "INVALIDTOKEN";
        case ENDOFFILE: return "ENDOFFILE";
        case NEWLINE: return "NEWLINE";
        case NUMBER: return "NUMBER";
        case IDENT: return "IDENT";
        case STRING: return "STRING";
        case LABEL: return "LABEL";
        // etc
```

This process is a lot more manual than I had anticipated and the way I am going about it at least, seems to be prone to error. Inevitably I'll add a new enum and forget to add it to one of the corresponding methods.


## Objects

Want to initialize an object as Null or None like you might do in Python on Java?

```java
public class MyObject {}

public class Main {
    public static void main(String[] args) {
        MyClass myObject = null;
    }
}
```

You can't do it this way at all. You need to initialize it as a null pointer, so you're dealing with a reference to the object.

```c++
class MyClass {};

int main() {
    MyClass* my_object = nullptr;
```

I found the easiest way to accomplish this is a member initializer list https://en.cppreference.com/w/cpp/language/constructor. In my case it looked something like this.

```c++
    Parser(Lexer lexer, Emitter emitter) : _lexer(lexer), _emitter(emitter), _currentToken(), _peekToken() {
        _symbols = {};
		_labelsDeclared = {};
		_labelsGotoed = {};
		
		nextToken();
        nextToken();
    }
```

Lexer and emmiter are initialized with passed in args, but the current token and the peek token are getting set later. I'd normally just set that to Null, but since I can't I either need to use a null pointer reference or I just use the member initializer list and it's doing everything I need it to.

I'm not going to lie, I don't really understand what's going on here, why this works, and I'll admit that when it compiled and ran it felt a little like magic. I can't play C++ but, I'm going to keep trying.

## Sets

Sets. Checking if an element is in a set seems so unnecessarily complicated when compared to python. In python you may simply just do something like:

```python
if "myemelent" in mySet:
    print("we found it")
```

Not so in C++. For an unordered set, you use the method `find("myElement")` this method returns an iterator pointing to the element. If the iterator is not pointing to the end of the set, then you've found the element you're looking for. So it looks more like:

```c++
if (mySet.find("myElement") != mySet.end()) {
    cout << "we found it";
}
```

This is fine, but it's a little confusing the first few times because you have to realize that having the `!=` means that you've found something that matches your element and `== mySet.end()` means that you haven't found what you're looking for. This sort of feels like the opposite of every other language I've used so I needed to remap my brain a little bit.

## Thoughts

I'm not quite done learning C++, in fact even after finishing the transpiler, I feel that I'm still nowhere near proficient in the language. It reminds me a lot of when I was trying to learn how to code in rust. I felt like was relearning how to program, and I feel that way a lot about c++. Many established patterns that I used in OO programming get thrown out the window. It has it's own advantages, but so far only made me appreciate all the niceties of Java and Python that I've come to enjoy.