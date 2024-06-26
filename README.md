如果能帮助到你，就请你给一颗[star](https://github.com/chenxing640/grab_ethtoken_info)，谢谢！(If this can help you, please give it a [star](https://github.com/chenxing640/grab_ethtoken_info), thanks!)

[![License MIT](https://img.shields.io/badge/license-MIT-green.svg?style=flat)](LICENSE)&nbsp;
[![Support](https://img.shields.io/badge/support-iOS%20|%20Android-blue.svg?style=flat)](https://pub.flutter-io.cn)&nbsp;

## grab_ethtoken_info

从以太坊区块链 (Ethereum Blockchain) etherscan 上抓取任意一个钱包地址的所有token信息 (Address, Name, Balance, Symbol, Value)，并编写界面进行展示。

## Group(ID:155353383) 

<div align=left>
&emsp; <img src="https://github.com/chenxing640/grab_ethtoken_info/raw/master/images/qq155353383.jpg" width="30%" />
</div>

## Recommendation

* [Dart crypto](https://github.com/chenxing640/dart_crypto)

## Getting Started

For help getting started with Flutter, view the online <br />

* [Flutter Hub](https://github.com/chenxing640/Awesome/blob/master/Flutter.md)

## Structure

- Network
  - [http_utils](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/network/http_utils.dart) - A HTTP tool to receive data, download and upload files, etc.

- DB
  - [EthTokenholdingsProvider](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/db/EthTokenholdingsProvider.dart)
  - [EthTokensProvider](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/db/EthTokensProvider.dart)

- Utils
  - [HtmlConverter](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/HtmlConverter.dart) - Converts a paragraph of HTML text into simple standard html.
  - [ExtendedUtility](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/ExtendedUtility.dart) - Extended Utilities.

- ETH Token
  - [AppConfigurator](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/AppConfigurator.dart)
  - [EthTokensDisplay](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokensDisplay.dart) - Display UI for tokens.
  - [EthAddNewTokens](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthAddNewTokens.dart) - Display UI for Adding new tokens.
  - [EthTokenholdingsDataRequest](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokenholdingsDataRequest.dart) - Get data from tokenholdings service.
  - [EthTokenholdingsModel](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokenholdingsModel.dart) - A model for the token holdings.
  - [EthTokenholdingsParser](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokenholdingsParser.dart) - Parsing data that comes from tokenholdings service.
  - [EthTokenModel](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokenModel.dart)
  - [EthTokenParser](https://github.com/chenxing640/grab_ethtoken_info/blob/master/lib/eth_token_grab/EthTokenParser.dart)

## Display UI

- Import Dart Files.

```dart
import './network/http_utils.dart';
import './eth_token_grab/EthTokensDisplay.dart';
import './eth_token_grab/EthTokenParser.dart';
import './db/EthTokensProvider.dart';
import './db/EthTokenholdingsProvider.dart';
import './eth_token_grab/EthTokenholdingsDataRequest.dart';
import './eth_token_grab/EthTokenholdingsParser.dart';
import './eth_token_grab/AppConfigurator.dart';
```

- Open Databases.

```dart
void openTokenholdingsDB() async { 
    EthTokenholdingsProvider.shared.open().then<bool>((isOpen){
        print("EthTokenholdingsProvider isOpen: $isOpen");
    });
}

void openTokensDB() async {
    EthTokensProvider.shared.open().then<bool>((isOpen) {
        print("EthTokensProvider isOpen: $isOpen");
    });
}

class MyApp extends StatelessWidget {
    // This widget is the root of your application.
    @override
    Widget build(BuildContext context) {
        openTokenholdingsDB();
        openTokensDB();

        return new MaterialApp(
            title: 'Flutter Demo',
            debugShowCheckedModeBanner: false,
            theme: new ThemeData(
                primarySwatch: Colors.blue,
            ),
            home: new MyHomePage(title: 'Flutter Demo Home Page'),
        );
    }
}
```

- Fetcth data, Create page route and Display the information of all tokens.

```dart
class _MyHomePageState extends State<MyHomePage> {
    int _counter = 0;
    String _data = "";
    
    // Fetch token data.
    void _getTokenData() {
        var baseUrl = "https://etherscan.io";
        var httpUtils = new HttpUtils.config(baseUrl: baseUrl);
        httpUtils.connectTimeout = 10;
        httpUtils.get("/address/0x71c7656ec7ab88b098defb751b7401b5f6d8976f");
        print("url: ${httpUtils.url}");
        httpUtils.listen((Response response) {
            String data = response.data;
            //print("data: $data");
            setState(() {
                _data = data;
            });
        }, (DioError error) {
            print("e: ${error.message}");
        });
    }

    // Fetch token holdings data.
    void _getTokenholdingsData() async {
        String a = "0x71c7656ec7ab88b098defb751b7401b5f6d8976f";
        String data = await EthTokenholdingsDataRequest.init(a).getData();
        if (data != null && data.isNotEmpty) {
            EthTokenholdingsParser.parse(data, a);
        }
    }

    @override
    void initState() {
        super.initState();
        _getTokenData();
        _getTokenholdingsData();
    }

    void _incrementCounter() {
        setState(() {
           // This call to setState tells the Flutter framework that something has
           // changed in this State, which causes it to rerun the build method below
           // so that the display can reflect the updated values. If we changed
           // _counter without calling setState(), then the build method would not be
           // called again, and so nothing would appear to happen.
           _counter++;
        });
        if (_data.isNotEmpty) {
            EthTokenParser.parse(_data).then<bool>((finished){
                if (finished) {
                    Navigator.push(context, _createPushRoute());
                }
            });
        } else {
            print("data is empty.");
        }
    }

    // Create page route for displaying the information of all of tokens.
    // This is simple demo for testing, the codes maybe include some bugs. 
    MaterialPageRoute _createPushRoute() {
        return MaterialPageRoute(builder: (context) => 
            new EthTokensDisplayView("0x71c7656ec7ab88b098defb751b7401b5f6d8976f")
        );
    }

    @override
    Widget build(BuildContext context) {
        return new Scaffold(
            appBar: new AppConfigurator('UI').createAppBar(
                title: new Text(widget.title, style: new TextStyle(fontSize: 18.0),)
            ),
        
            body: new Center(
                // Center is a layout widget. It takes a single child and positions it
                // in the middle of the parent.
                child: new Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                        new Text(
                            'You have pushed the button this many times:',
                        ),
                        new Text(
                            '$_counter',
                            style: Theme.of(context).textTheme.display1,
                        ),
                    ],
                ),
            ),
            
            floatingActionButton: new FloatingActionButton(
                nPressed: _incrementCounter,
                tooltip: 'Increment',
                child: new Icon(Icons.add),
            ), // This trailing comma makes auto-formatting nicer for build methods.
        );
    }
}
````

## Feedback is welcome

If you notice any issue, please create an issue. I will be happy to help you.
