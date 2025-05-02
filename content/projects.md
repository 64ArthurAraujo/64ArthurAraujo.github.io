+++
title = "Projects"
slug = "projects"
+++

---

# [WordView](https://github.com/word-view) (2023 - Now)
A language learning platform centered around music and currently, my main focus.

The idea is for the user to select any international song and while
he is listening, the app will identify relevant words in the lyrics
and generate, at the end, a lesson based on them.

Besides the music part, there are also plans to create another
section based on books.

## The structure
There are two main projects: APIWordView, and WordView-android. And three auxiliary libraries: Gengolex, Assis, and WordFind.


###### [APIWordView](https://github.com/word-view/APIWordView)
An REST API created with spring boot using PostgreSQL as it's database, it manages lyrics, lessons, images and account authentications, etc. There are some adaptions made to the usual way a spring boot 
api is structured that makes development faster and the code easier to understand.

###### [WordView-android](https://github.com/word-view/WordView-android)
Currently, the only front-end of the project. The app is created using Kotlin with the Jetpack Compose toolkit. Currently it has a basic home with some music recommendations, a search bar, where you can look for any songs, a login/register system, and the player where you can listen to the music and read the lyrics.

There are some plans in the future to convert it into a Kotlin Multiplatform project, allowing it to run on Android, IOS, Web and locally in the PC using the same codebase.

You can see some images of the app at the [gallery](gallery/#wordview-android).

###### [Gengolex](https://github.com/word-view/gengolex)
Language parser and lexer written in Kotlin. To use it, the user has to create a new `Parser` with the desired language
supported by the library, and either use a dictionary defined in the resources folder or a runtime dictionary. The dictionary contains the registration of the words and their characteristics such as the way that the word can be displayed, if it is a verb or a noun, if the verb is present past or future, etc. Each word has an parent, that is ideally the translation of the word in english, an example can be seen [here](https://github.com/word-view/gengolex/blob/master/src/test/resources/dictionaries/kanji/kanji.json).

With everything set up, the user can parse a string of characters using `Parser.findWords`, this function returns a list of the words contained in the string that are defined in the dictionary.

###### [Assis](https://github.com/word-view/assis)
Assis (named after Brazilian writer Machado de Assis) is an ebook reader written in Kotlin. Currently unused, but will be in the future for the book section of the app. 

It can parse most epub files, although it is not very stable yet. To use it the user can call `parseEpub` and pass the epub file InputStream as the argument. The returned value is a EpubBook class that has some methods to get the title, cover and pages text.

###### [WordFind](https://github.com/word-view/WordFind)
WordFind is a library to search for lyrics written in Kotlin, currently it searches in three different platforms, but can be extended by the library user. 

It has a very simple and straight-forward structure, every class that extends `Provider` has a `rootUrl` and a function called `find`, it's task is to build the request, injecting `rootUrl` with the search parameters and execute it. The request response passed to `parseResponse` that decides what to do based on the returned status code, if it is 200 OK, the raw lyrics are extracted from the response JSON and returned to the main class.

To add a custom platform, the user needs to create his own subclass of `Provider` and insert it at the ArrayList `WordFind.platforms`, the library then loops through all the items inside this list until it finds a platform that returns a valid lyrics response.


# [Quernel](https://github.com/64ArthurAraujo/Quernel) (2023)
A simple i686 kernel i created to learn C and Assembly language. It includes a simple libc as well.