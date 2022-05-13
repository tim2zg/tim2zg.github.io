---
layout: post
title: "The location pain in flutter."
date: 2022-05-13
categories: programming
tags: flutter tim2zg dart location geolocator
---
# The location pain in flutter.


So, a friend and I try to get the user’s location in flutter, and in total, we wasted a lot of time (approx. 4h). And yes, we just could be completely lost, at this point I just don’t care anymore…
So here is the code:

Dependencies:

`geolocator: ^8.2.1`

Don't forget to add

`<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />`

`<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />`

to the `AndroidManifest.xml`!

```
Future<bool> request_permissions() async {
    LocationPermission permission;
    permission = await Geolocator.requestPermission();

    if (permission == LocationPermission.denied) {
      return false; // Alert with Popup!
    } else if (permission == LocationPermission.deniedForever) {
      return false; // Alert with Popup!
    } else {
      return true;
    }
  }

  Future<bool> get_location_perms() async {
    bool serviceenabled;
    LocationPermission permission;

    serviceenabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceenabled) {
      // Alert with Popup
      print("Service not enabled!");
      return false;
    }

    permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.whileInUse) {
      return true;
    } else if (permission == LocationPermission.always) {
      return true;
    } else {
      return request_permissions();
    }
  }

  Future<Position?> get_position() async {
    if (await get_location_perms() == true) {
      Geolocator.getCurrentPosition(forceAndroidLocationManager: true, timeLimit: const Duration(seconds: 10));
      print(await Geolocator.getCurrentPosition()); // Web
      print(await Geolocator.getLastKnownPosition(forceAndroidLocationManager: true)); // Android
      return await Geolocator.getCurrentPosition();
    }
  }
```



“It ShOuLd bE sElF-eXpLaNaToRy”
To get a position we first call the get permission function.
If we have, we return true. If not we try to get the permission from the user with the request permission function. If he refuses, we return a false and print an error message.

Here is a full sample:

```
import 'package:flutter/material.dart';
import 'package:geolocator/geolocator.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        // This is the theme of your application.
        //
        // Try running your application with "flutter run". You'll see the
        // application has a blue toolbar. Then, without quitting the app, try
        // changing the primarySwatch below to Colors.green and then invoke
        // "hot reload" (press "r" in the console where you ran "flutter run",
        // or simply save your changes to "hot reload" in a Flutter IDE).
        // Notice that the counter didn't reset back to zero; the application
        // is not restarted.
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key, required this.title}) : super(key: key);

  // This widget is the home page of your application. It is stateful, meaning
  // that it has a State object (defined below) that contains fields that affect
  // how it looks.

  // This class is the configuration for the state. It holds the values (in this
  // case the title) provided by the parent (in this case the App widget) and
  // used by the build method of the State. Fields in a Widget subclass are
  // always marked "final".

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  Future<bool> request_permissions() async {
    LocationPermission permission;
    permission = await Geolocator.requestPermission();

    if (permission == LocationPermission.denied) {
      return false; // Alert with Popup!
    } else if (permission == LocationPermission.deniedForever) {
      return false; // Alert with Popup!
    } else {
      return true;
    }
  }

  Future<bool> get_location_perms() async {
    bool serviceenabled;
    LocationPermission permission;

    serviceenabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceenabled) {
      // Alert with Popup
      print("Service not enabled!");
      return false;
    }

    permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.whileInUse) {
      return true;
    } else if (permission == LocationPermission.always) {
      return true;
    } else {
      return request_permissions();
    }
  }

  Future<Position?> get_position() async {
    if (await get_location_perms() == true) {
      Geolocator.getCurrentPosition(forceAndroidLocationManager: true, timeLimit: const Duration(seconds: 10));
      print(await Geolocator.getCurrentPosition()); // Web
      print(await Geolocator.getLastKnownPosition(forceAndroidLocationManager: true)); // Android
      return await Geolocator.getCurrentPosition();
    }
  }

  @override
  Widget build(BuildContext context) {
    // This method is rerun every time setState is called, for instance as done
    // by the _incrementCounter method above.
    //
    // The Flutter framework has been optimized to make rerunning build methods
    // fast, so that you can just rebuild anything that needs updating rather
    // than having to individually change instances of widgets.
    return Scaffold(
      appBar: AppBar(
        // Here we take the value from the MyHomePage object that was created by
        // the App.build method, and use it to set our appbar title.
        title: Text(widget.title),
      ),
      body: Center(
        // Center is a layout widget. It takes a single child and positions it
        // in the middle of the parent.
        child: Column(
          // Column is also a layout widget. It takes a list of children and
          // arranges them vertically. By default, it sizes itself to fit its
          // children horizontally, and tries to be as tall as its parent.
          //
          // Invoke "debug painting" (press "p" in the console, choose the
          // "Toggle Debug Paint" action from the Flutter Inspector in Android
          // Studio, or the "Toggle Debug Paint" command in Visual Studio Code)
          // to see the wireframe for each widget.
          //
          // Column has various properties to control how it sizes itself and
          // how it positions its children. Here we use mainAxisAlignment to
          // center the children vertically; the main axis here is the vertical
          // axis because Columns are vertical (the cross axis would be
          // horizontal).
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '',
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: get_position,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}

```

You can find the whole code in this [repo](https://github.com/tim2zg/blog_samples).

Yeah I really have to improve the example repo!




