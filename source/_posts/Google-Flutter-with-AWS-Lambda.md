---
title: Google Flutter with AWS Lambda
date: 2018-03-26 12:59:40
tags:
- Android
- Dart
- Flutter
- Lambda
categories:
- Mobile
cover_index: /assets/Google-Flutter-with-AWS-Lambda-Index.png
---
![](https://cdn-images-1.medium.com/max/1000/1*xTO44D46BBr4WkIAStzjJw.png)

Google recently announced the beta release of Flutter at [Mobile World Congress 2018](https://developers.googleblog.com/2018/02/announcing-flutter-beta-1.html). Flutter is new open-source mobile UI framework that’s used to build native apps for both iOS and Android.

![](https://cdn-images-1.medium.com/max/800/1*GRHjdlq4MidsE8bpvFnGrQ.png)

Flutter uses Dart to write applications, so the syntax should look very familiar if you already know Java, JavaScript, C#, or Swift. The code is compiled using the standard Android and iOS toolchains for the specific mobile platform — hence, much better performance and startup times.

Flutter has a lot of benefits for developers:

* The project is open-source
* Hot reloads for quicker development cycle
* Native ARM code runtime
* Rich widget set & growing [community of plugins](https://pub.dartlang.org/flutter) backed by Google
* Excellent editor integration with Android Studio & Visual Studio Code
* Single codebase for iOS and Android with full native performance that *does not* use JavaScript as a bridge or WebViews
* Reusable components similar to React Native
* Dart feels like Java — making it easy for many developers to jump in
* The toolkit use Widgets, so everything should seem very familiar for anyone coming from a web development background
* This might end the Java holy wars between Google vs Oracle

Creating a application using Flutter was a great opportunity to get my hands dirty and evolve the** **Serverless Golang API demonstrated in my previous post — “[Serverless Golang API with AWS Lambda](https://read.acloud.guru/serverless-golang-api-with-aws-lambda-34e442385a6a)”.

The movie website uses an AWS Lambda function written in Go — the latest language for serverless applications. The site retrieves and displays the latest movie releases in your browser.

![](https://cdn-images-1.medium.com/max/800/1*F8yHKVEd-3_WjWerGK8CmQ.png)

Now let’s create a mobile version of our movie site using Flutter!

#### Building a serverless mobile app using Flutter

The Flutter application will call the API Gateway, and then invoke an AWS Lambda function. The function will use the TMDB API to retrieve a list of movies airing this week in theaters — all in real-time. The application will consume the JSON response and then display results in a ListView.

*Note: All code can be found in my *[GitHub](https://github.com/mlabouardy/flutter-watchnow)* repository.*

To get started, follow the previous tutorial on how to create a [Serverless API](https://read.acloud.guru/serverless-golang-api-with-aws-lambda-34e442385a6a).

* Once it’s deployed, copy the `API Gateway Invoke URL` to the clipboard.
* Next, download the Flutter SDK** **by cloning the following GitHub repository:`git clone -b beta https://github.com/flutter/flutter.git`
* Make sure to add `flutter/bin` folder to your `PATH` environment variable.
* Next, install the missing dependencies and SDK files: `flutter doctor`
* Start Android Studio and install the Flutter plugin from *File>Settings>Plugins*

![](https://cdn-images-1.medium.com/max/800/0*4_oFg0vFTXqv5ieg.)

* Now it’s time to create a new Flutter project using Android Studio.
* Since Flutter also comes with a CLI, you can also create a new project with the following command: `flutter create PROJECT_NAME`.

![](https://cdn-images-1.medium.com/max/800/0*eC0dGnn6Mkh8OLy5.)

* Android Studio will generate the files for a basic Flutter sample app.
* We will be doing our work in the `lib/main.dart` file:

![](https://cdn-images-1.medium.com/max/800/0*n1Wgh5CSMd4x_E5Z.)

* Now you can start the app — the following screen should appear:

![](https://cdn-images-1.medium.com/max/800/0*Ft74NiKFXl5aq0bo.)

* Next, create a simple POJO class named `Movie`** **using the following set of attributes and getters:

{% gist 4cd90126b3d7fb7c13e1b2d8cfe20e11 movie.dart %}

* Now create a widget named`TopMovies`.
* That step will create its state called `TopMoviesState`.
* The state class will maintain the list of movies.
* Next, add the stateful `TopMovies` widget to *main.dart*:

{% gist aa5157fb6c37abd3cb998be6cf11e1a7 movie_widget.dart %}

* Now it’s time to add the `TopMoviesState` class.
* Most of the app’s logic will resides in this class.

{% gist 90363dd67229f16baa1bd1fc90434971 movie_state.dart %}

* Let’s initialize our `_movies` variable with a list of movies by invoking API Gateway. We will iterate through the JSON response, and add the movies using the `_addMovie` function:

{% gist fe9fb672f84bff608db7a5af73dbed3d main.dart %}

* The `_addMovie()` function will simply add the movie passed as an argument to list of `_movies`:

{% gist e02b9781b9642d9a8cfd76859ce2f501 main.dart %}

* Now we’ll need to display movies in a scrolling `ListView`**.** Flutter** **comes with a suit of powerful basic widgets to make this easy.
* In the code below, I used the `Text`, `Row`, and `Image` widgets.
* The `Padding` and `Align` components properly display a movie item.

{% gist 8a29da1cf82b32f11a80841b33b1a869 main.dart %}

* Finally, update the build method for `MyApp` to call the `TopMovies` widget:

{% gist 29927726042ae5a9423b8d69b46b1c25 main.dart %}

* Restart the app —a list of current movies should appear!

![](https://cdn-images-1.medium.com/max/800/0*dIgfT5BE-4Q8XHC0.)

#### And the Oscar goes to …

Congratulations! We’ve successfully created a serverless application *in just 143 lines of code — *and it works like a charm on both Android and iOS.

Flutter is still in the early stages — so of course it has some drawbacks:

* Steep learning curve compared to React Native which uses JavaScript
* Dart is unpopular compared to Kotlin or Java
* Does not support 32-bit iOS devices
* Due to autogenerated code, the** **build artifact is *hug**e. The *APK for
Android is almost 25 Mb — while I built the same app in Java for 3 Mb.

For an beta open-source project, it look very stable and well designed. I can’t wait to see what you can build with Flutter!