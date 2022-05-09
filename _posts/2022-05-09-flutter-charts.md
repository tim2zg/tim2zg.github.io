---
layout: post
title: "Flutter Charts with InfluxDB"
date: 2022-05-09
categories: programming
tags: flutter tim2zg dart influx influxdb charts
---
# Flutter Charts tutorial with Influx as a data source.

## What do we need:
Influx DB Version 1. X


First, we add the dependencies:

`charts_flutter: ^0.12.0`

`influxdb_client: ^2.2.0`

Because if we query data from the DB, it should happen asynchronous (at the same time in the background). We need to build the Charts with a Future builder.
```
import 'package:charts_flutter/flutter.dart' as charts;
import 'package:flutter/material.dart';


class datachart extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return datachartwidget(); // Calling the Chart widget
  }

}

class datachartwidget extends State<datachart> { // Our Chart widget
  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
        future: // Function that gets the Data
        builder: (BuildContext context, AsyncSnapshot snapshot) {
          if (snapshot.connectionState == ConnectionState.done) { // See if the data is there
            return charts.TimeSeriesChart( // If data is there we return the chart
              snapshot.data,
              animate: true,
              customSeriesRenderers: [
                charts.SymbolAnnotationRendererConfig(
                    customRendererId: 'customSymbolAnnotation')
              ],
              dateTimeFactory: const charts.LocalDateTimeFactory(),
            );
          } else {
            return const Center(child: CircularProgressIndicator());// Else we return a Progressbar
          }
        }
    );
  }
}
```

We made a constructor which will return a time chart as soon as the function returns the data. But we first must make such a function that queries the data from the DB.
```
import 'package:influxdb_client/api.dart';
import 'package:charts_flutter/flutter.dart' as charts;

Future<List<charts.Series<dynamic, DateTime>>> getdata(String duration, String timespann) async {
  var thedata = [];

  InfluxDBClient client = InfluxDBClient(
      url: '', // Your URL for Example: http://127.0.0.1:8086
      username: '',  // Your Username of the DB for Example: admin
      password: '', // Your Password of the DB for Example: 123
      debug: false);

  var queryService = client.getQueryService();

  var query = await queryService.query('''from(bucket: "iobroker")
  |> range(start: -${duration})
  |> filter(fn: (r) => r["_measurement"] == "0_userdata.0.SolarDaten.Eigenverbrauch_%")
  |> filter(fn: (r) => r["_field"] == "value")
  |> aggregateWindow(every: ${timespann}, fn: mean, createEmpty: false)
  |> yield(name: "mean")''');

  await query.forEach((element) {
    thedata.add(data(DateTime.parse(element['_time']), element['_value']));});

  client.close();

  return
    [charts.Series<dynamic, DateTime>(
      id: 'Eigenverbauch %',
      colorFn: (_, __) => charts.MaterialPalette.blue.shadeDefault,
      domainFn: (data, _) => data.time,
      measureFn: (data, _) => data.value,
      data: thedata,
    ),
    ];
}

class data {
  final DateTime time;
  final double value;

  data(this.time, this.value);
}
```


So we made a function that returns ``charts.Series``. First, we need to create a Database Client, it takes a Username, a Password, and the Host. Then we write a query in the flux language. You can use the example provided, but make sure you query the Data from the right Database (just replace ``ioBroker`` with the name of the Database) and the right value (just replace the ``0_userdata.0.SolarDaten.Eigenverbrauch_%`` with your time series). At the start, we define a data class that allows us to map the time value to a double value for our time series chart. Then we take the incoming data and map it to our data class, which gets added to the ``thedata`` variable. We close the client. And return a list with our one-time series chart that gets our data.

I would write the getdata function in another file and import it:

``charts.dart``:

```
import 'package:charts_flutter/flutter.dart' as charts;
import 'package:flutter/material.dart';
import 'getdata.dart';


class datachart extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return datachartwidget(); // Calling the Chart widget
  }

}

class datachartwidget extends State<datachart> { // Our Chart widget
  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
        future: getdata("1d", "1m"), // Function that gets the Data
        builder: (BuildContext context, AsyncSnapshot snapshot) {
      if (snapshot.connectionState == ConnectionState.done) { // See if the data is there
        return charts.TimeSeriesChart( // If data is there we return the chart
          snapshot.data,
          animate: true,
          customSeriesRenderers: [
            charts.SymbolAnnotationRendererConfig(
                customRendererId: 'customSymbolAnnotation')
          ],
          dateTimeFactory: const charts.LocalDateTimeFactory(),
        );
      } else {
        return const Center(child: CircularProgressIndicator());// Else we return a Progressbar
      }
    }
    );
  }
}
```
``getdata.dart``:

```
import 'package:influxdb_client/api.dart';
import 'package:charts_flutter/flutter.dart' as charts;

Future<List<charts.Series<dynamic, DateTime>>> getdata(String duration, String timespann) async {
  var thedata = [];

  InfluxDBClient client = InfluxDBClient(
      url: '', // Your URL for Example: http://127.0.0.1:8086
      username: '',  // Your Username of the DB for Example: admin
      password: '', // Your Password of the DB for Example: 123
      debug: false);

  var queryService = client.getQueryService();

  var query = await queryService.query('''from(bucket: "iobroker")
  |> range(start: -${duration})
  |> filter(fn: (r) => r["_measurement"] == "0_userdata.0.SolarDaten.Eigenverbrauch_%")
  |> filter(fn: (r) => r["_field"] == "value")
  |> aggregateWindow(every: ${timespann}, fn: mean, createEmpty: false)
  |> yield(name: "mean")''');

  await query.forEach((element) {
    thedata.add(data(DateTime.parse(element['_time']), element['_value']));});

  client.close();

  return
    [charts.Series<dynamic, DateTime>(
      id: 'Eigenverbauch %',
      colorFn: (_, __) => charts.MaterialPalette.blue.shadeDefault,
      domainFn: (data, _) => data.time,
      measureFn: (data, _) => data.value,
      data: thedata,
    ),
    ];
}

class data {
  final DateTime time;
  final double value;

  data(this.time, this.value);
}
```

This requires importing the other file into the ``charts.dart`` as seen at the top. Additionally, you hat so add the time span and the frequency. For Example, all the data from a day and a value per Minute. With these parameters, you can control the bandwidth and the tress of your database.

I hope it help you and have a nice day!
