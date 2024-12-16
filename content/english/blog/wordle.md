---
title: "How to build a Wordle with Flutter"
meta_title: "How to build a Wordle with Flutter"
description: "How to build a Wordle with Flutter"
date: 2022-01-14T05:00:00Z
image: "/images/blog/wordle/wordle1.png"
categories: ["POC"]
author: "Fernando Souto"
tags: ["wordle", "Flutter", "Game", "Challenge"]
draft: false
---

I saw last weekend some people sharing random squares on Twitter and I was thinking…
>  What is it?

![My friend Camilo](/images/blog/wordle/camilo.png)

After researching a bit, I found a post explaining that this is a game called **Wordle** and it is a super awesome game.

**Kudos for the dev.**

You have to guess one word per day.

The idea of Having to enter one word per day is like the old times in the internet. The developer behind **Wordle** is not trying to push our limits to play like addicts and thats **super nice**.
>  This “one time” thing smells like internet before algorithms and social networks.

And after playing a couple of days I was wondering:
>  Can I build something similar in a small amount of hours with Flutter?

I push my limits to create the clon of **Wordle** in less than 4 hours with the awesome **Flutter** dev tools.

![Wordle with Flutter](/images/blog/wordle/wordle1.png)

## The game

I have a matrix of 5x6 for the colours and the characters and every time the user inputs a character with the virtual keyboard I put the character in the correct place.

![The matrix](/images/blog/wordle/wordle2.png)

When the user reaches the end of the row then the user can DELETE a character or click ENTER to verify the current word. If the user decides to press ENTER then I execute the algorithm to verify *the input vs the solution* with the following code(maybe not the best one but I was rushed to complete the task):

![Game algorithm to verify the solution](/images/blog/wordle/wordle3.png)

The rest of the code is for building the UI.

In the following code we are building the top part of the UI with the boxes that the user needs to input the characters.

![User input boxes](/images/blog/wordle/wordle4.png)

For the *keyboard* I rebuilt a bit a library called *virtual_keyboard_multi_language.*

Its a simple column with 3 rows with some custom decoration to show the appearance of the **Wordle** keyboard, some arrays with the values for each position and thats it!

![Virtual keyboard](/images/blog/wordle/wordle5.png)

![Values of each element of the rows](/images/blog/wordle/wordle6.png)

If you didn’t try the original game yet. Please do it:
[**Wordle - A daily word game**
*Guess the hidden word in 6 tries. A new puzzle is available each day.* www.powerlanguage.co.uk](https://www.powerlanguage.co.uk/wordle/)

This is the source code of my small copy of the game:
[**GitHub - ficiverson/wordle-flutter**
*You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…*github.com](https://github.com/ficiverson/wordle-flutter)

I don’t want to *copy the idea or have a commercial version* of the game but I saw a guy that published into the app to **Appstore** the game without the permission of the original devs and **that’s not nice.**

![Copy guy](/images/blog/wordle/copy.png)

All the credits of the idea for the **original authors. **Thanks :)
