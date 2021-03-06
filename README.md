# React Native Starter Kit (for Android only so far)

This starter kit includes all of the dependencies you will need to persist documents to a Couchbase Lite database in a React Native project. You will find the following:

- React Native
- Couchbase Lite
- React Native Module to interface with the Couchbase Lite database
- Initial Couchbase Lite REST API wrapper for Fetch

## Prerequisites

- Node.js 4.0 or higher
- [Android Studio](http://developer.android.com/sdk/installing/studio.html) and an emulator

## Getting Started

Just clone the repo and start coding:

```
$ git clone https://github.com/couchbaselabs/reactnative-starter-kit.git
$ cd reactnative-starter-kit
$ cd android
$ npm install
$ npm start
```

To install the app on the emulator:

```
$ cd android
$ ./gradlew installDebug
```

Watch this [short screencast](https://www.youtube.com/watch?v=ajHughXQ5Yw) for more debugging tips.

## Sync Gateway

Download Sync Gateway from the link below and unzip the file:

> http://www.couchbase.com/nosql-databases/downloads

In a new file named **sync-gateway-config.json**, paste the following:

```js
{
  "log": ["*"],
  "databases": {
    "todos": {
      "server": "walrus:",
      "users": { "GUEST": { "disabled": false, "admin_channels": ["*"] } }
    }
  }
}
```

And run Sync Gateway with this config file:

```js
~/Downloads/couchbase-sync-gateway/bin/sync_gateway /path/to/project/sync-gateway-config.json
```

To make the Sync Gateway endpoint reachable inside of the Android VM emulator, you need to enable a port from the host to the VM. In Terminal, run the following:

```bash
adb reverse tcp:4984 tcp:4984
```

Open the Admin UI to monitor the documents that were saved to Sync Gateway:

> http://localhost:4985/_admin/

## CBLModule

In order to make the Couchbase Lite functionalities accessible from the React Native application written in JavaScript, we've added a Native Module called `CBLModule` to start the Couchbase Lite Listener with a random username and password on port `5984`. You can find out more about how Native Modules work by reading [this part](http://facebook.github.io/react-native/docs/native-modules-android.html#callbacks) of the React Native documentation. To get the Couchbase Lite URL asynchronously, you can simply use the `getUrl` method from the `CBLModule`. With that URL, use [fetch](https://facebook.github.io/react-native/docs/network.html) to send any network request to the database. You can extend the basic wrapper for the Couchbase Lite REST API already available in this project:

```js
var React = require('react-native');
var couchbase = require('./../utils/couchbase-lite-fetch');
var CBLModule = require('./../utils/CBLModule');

var {
  Text,
  View,
  } = React;

class Home extends React.Component {
  componentWillMount() {
    CBLModule.getUrl((err) => {
      console.log('Callback error :: ' + err);
    }, (url) => {
      var db = new couchbase(url, "todos");
      db.createDatabase()
        .then((res) => {
          db.createDocument({"text": "hello there"})
            .then((res) => {
              console.log(res);
            });
          return res;
        });
    });
  }
  
  render() {
    return (
      <View>
        <Text>Hello</Text>
      </View>
    );
  }

}

module.exports = Home;
```



Read more about the available endpoints on the REST API in the [documentation](http://developer.couchbase.com/documentation/mobile/1.1.0/develop/references/couchbase-lite/rest-api/index.html).