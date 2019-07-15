# WebSocket API (Market Data) 

## Start use

WebSocket  protocol which achieves full-duplex communications over a  single TCP connection between client and server is based on a new network protocol of the TCP.Server sends the  information to the client,reducing unnecessary overhead such as frequent authentication.Its biggest advantage is that：

> - Request header is small in size (around 2 bytes) .
> - The server no longer passively returns the data after receiving the client's request, but actively pushes the new data to the client.
> - There is no need to create and  delete TCP connection repeatedly,it saves bandwidth and server resources.



## Request & subscription instruction

* Access to the address: `wss://www.lbkex.net/ws/V2/`

* Hearbeat（ping/pong）

    To prevent botch links and reduce server load，the server will send heartbeat periodically to the client connections.When client recieves this heartbeat message,it should response immediately.if you do not respond to any macthing "Ping"  for more than a minute,the server will disconnect this client.And then client can also send a "Ping" message to  the server  to check whether the  connection is working.After the server recieves “Ping”message ，it should response with a matching "pong" message.

    **Eg:**

    ```javascript
    
    # ping
    {
        "action":"ping",
        "ping":"0ca8f854-7ba7-4341-9d86-d3327e52804e"
    }
    # pong
    {
        "action":"pong",
        "pong":"0ca8f854-7ba7-4341-9d86-d3327e52804e"
    }
    ```

    When client receives this heartbeat message, it should response with a matching "pong" message which has the same integer in it.



* Subscription/Unsubscription

    Each subscription data should include at least one subscribe field  specified by the data type of the subscription. Data that can now be subscribed includes：kbar，tick，depth，trade.Each subscription needs to a pair field to specify the subscribed trading pair,which is joined with underline(_).
    

    1.Subscription K-line Data

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |action|String|Is|Type of action requested:`subscribe`,`unsubscribe`|
    |subscribe|String|Is|`kbar`|
    |kbar|String|Is|To subscribe to k-line types<br>`1min`:1,min<br>`5min`:5min<br>`15min`:15min<br>`30min`:30min<br>`1hr`:1hr<br>`4hr`:4hr<br>`day`:1day<br>`week`:1week<br>`month`:1month<br>`year`:1year|
    |pair|String|Is|Trading pair:`eth_btc`|

    **Eg:**

    ```javascript
    
    # Subscription request
    {
        "action":"subscribe",
        "subscribe":"kbar",
        "kbar":"5min",
        "pair":"eth_btc"
    }
    # Push data
    {
        "kbar":{
            "a":64.32991311,
            "c":0.02590293,
            "t":"2019-06-28T17:45:00.000",
            "v":2481.1912,
            "h":0.02601247,
            "slot":"5min",
            "l":0.02587925,
            "n":272,
            "o":0.02595196
        },
        "type":"kbar",
        "pair":"eth_btc",
        "SERVER":"V2",
        "TS":"2019-06-28T17:49:22.722"
    }
    ```

    **Return value specification:**

    |Parameters|Parameters Type|Description|
    | :-----    | :-----  | :-----   |
    |t|Long|K-line updates the timestamp|
    |o|BigDecimal|Open price|
    |h|BigDecimal|Highest price|
    |l|BigDecimal|Lowest price|
    |c|BigDecimal|Close price|
    |v|BigDecimal|Trading volume|
    |a|BigDecimal|Aggregated trading value（in quote currency）|
    |n|BigDecimal|Number of trades|
    |slot|String|K-line type|


    2.Market Depth

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |action|String|Is|Type of action requested:`subscribe`,`unsubscribe`|
    |subscribe|String|Is|`depth`|
    |depth|String|Is|Pro-choise:10/50/100|
    |pair|String|Is|Trading pair:`eth_btc`|


    **Eg:**

    ```javascript

    # Subscription request
    {
        "action":"subscribe",
        "subscribe":"depth",
        "depth":"100",
        "pair":"eth_btc"
    }
    # Push data
    {
        "depth":{
            "asks":[
                [
                    0.0252,
                    0.5833
                ],
                [
                    0.025215,
                    4.377
                ],
                ...
            ],
            "bids":[
                [
                    0.025135,
                    3.962
                ],
                [
                    0.025134,
                    3.46
                ],
                ...
            ]
        },
        "count":100,
        "type":"depth",
        "pair":"eth_btc",
        "SERVER":"V2",
        "TS":"2019-06-28T17:49:22.722"
    }
    ```

    **Return value specification:**

    |Parameters|Parameters Type|Description|
    | :-----    | :-----  | :-----   |
    |asks|List|Selling side,list.get(0):Delegat price, list.get(1):Delegate quantity|
    |bids|List|Buying side|


    3.Trade record

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |action|String|Is|Type of action requested:`subscribe`,`unsubscribe`|
    |subscribe|String|Is|`trade`|
    |pair|String|Is|Trading pair:`eth_btc`|


    **Eg:**

    ```javascript

    # Subscription request
    {
        "action":"subscribe",
        "subscribe":"trade",
        "pair":"eth_btc"
    }
    # 推送数据
    {
        "trade":{
            "volume":6.3607,
            "amount":77148.9303,
            "price":12129,
            "direction":"sell",
            "TS":"2019-06-28T19:55:49.460"
        },
        "type":"trade",
        "pair":"btc_usdt",
        "SERVER":"V2",
        "TS":"2019-06-28T19:55:49.466"
    }
    ```

    **Return value specification:**

    |Parameters|Parameters Type|Description|
    | :-----    | :-----  | :-----   |
    |amount|String|Trading volume|
    |price|Integer|Trade price|
    |volumePrice|String|Aggregated trading value|
    |direction|String|`sell`,`buy`|
    |TS|String|Deal time|


    4.Market

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |action|String|Is|Type of action requested:`subscribe`,`unsubscribe`|
    |subscribe|String|Is|`tick`|
    |pair|String|Is|Trading pair:`eth_btc`|


    **Eg:**

    ```javascript

    # Subscription request
    {
        "action":"subscribe",
        "subscribe":"tick",
        "pair":"eth_btc"
    }
    # ush data
    {
        "tick":{
            "to_cny":76643.5,
            "high":0.02719761,
            "vol":497529.7686,
            "low":0.02603071,
            "change":2.54,
            "usd":299.12,
            "to_usd":11083.66,
            "dir":"sell",
            "turnover":13224.0186,
            "latest":0.02698749,
            "cny":2068.41
        },
        "type":"tick",
        "pair":"eth_btc",
        "SERVER":"V2",
        "TS":"2019-07-01T11:33:55.188"
    }
    ```

    **Return value specification:**

    |Parameters|Parameters Type|Description|
    | :-----    | :-----  | :-----   |
    |high|BigDecimal|24 hour high|
    |low|BigDecimal|24 hour low|
    |latest|BigDecimal|Last traded price|
    |vol|BigDecimal|Trading volume|
    |turnover|BigDecimal|Aggregated trading value（in quote currency）|
    |to_cny|BigDecimal|such as`eth_btc`, convert btc into cny|
    |to_usd|BigDecimal|such as`eth_btc`, convert btc into usd|
    |cny|BigDecimal|such as`eth_btc`, convert eth into cny|
    |usd|BigDecimal|such as`eth_btc`, convert eth into usd|
    |dir|String|`sell`,`buy`|
    |change|BigDecimal|24 hour price limit|



    **取Unsubscribe example:**

    ```javascript

    #Unsubscribe from k-line
    {
        "action":"unsubscribe",
        "subscribe":"kbar",
        "kbar":"5min",
        "pair":"eth_btc"
    }
    #Unsubscribe from deep subscription
    {
        "action":"unsubscribe",
        "subscribe":"depth",
        "depth":"100",
        "pair":"eth_btc"
    }
    ```


* Request data

    While connected to websocket, you can also use it in pull style by sending message to the server.

    1.Pull request is supported with extra parameters to define the range：

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |start|String|fasle|Start time ,Accept both formats，such as`2018-08-03T17:32:00`（beijing time）, another temestamp，such as`1533288720`(Accurate to seconds)|
    |end|String|false|deadline|
    |size|String|false|Gets the number of kbars|


    2.Pull request is supported with extra parameters to obtain trade records.

    **Parameter:**

    |Parameters|Parameters Type|Mandatory|Description|
    | :-----    | :-----   | :-----    | :-----   |
    |size|String|Is|获Gets the number of trades|


    **Eg:**

    ```javascript
    
    # Get k-line data Request
    {
        "action":"request",
        "request":"kbar",
        "kbar":"5min",
        "pair":"eth_btc"
        "start":"2018-08-03T17:32:00"
        "end":"2018-08-05T17:32:00"
        "size":"576"
    }
    # Get depth data Request
    {
        "action":"request",
        "request":"depth",
        "depth":"100",
        "pair":"eth_btc"
    }
    # Get transaction data Request
    {
        "action":"request",
        "request":"trade",
        "pair":"eth_btc"
        "size":"100",
    }
    # Get market data Request
    {
        "action":"request",
        "request":"tick",
        "pair":"eth_btc"
    }
    ```
      

