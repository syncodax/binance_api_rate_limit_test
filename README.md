# Binance API Rate Limit Test


## Official API rate limit
[Official API docs](https://github.com/binance-exchange/binance-official-api-docs/blob/master/rest-api.md#limits)
```
GET https://api.binance.com/api/v1/exchangeInfo
```

```javascript
"rateLimits":[
    {
        "rateLimitType": "REQUEST_WEIGHT",
        "interval": "MINUTE",
        "limit": 1200
    },
    {
        "rateLimitType": "ORDERS",
        "interval": "SECOND",
        "limit": 10
    },
    {
        "rateLimitType": "ORDERS",
        "interval": "DAY",
        "limit": 100000
    }
]
```

## Request - Symbol order book ticker
```
GET /api/v3/ticker/bookTicker
```
* **Weight:** 1

### Non-threaded

* Log: [binance_api_rate_limit_test_request_non-threaded.log](log/binance_api_rate_limit_test_request_non-threaded.log) 
* Interval between requests: 0ms
* Time taken to send 1200 requests and receive responses: 1m 5s

Error didn't occur because it was hard to exceed the rate limit(1200) within one minute in non-threaded mode.

### Threaded #1

* Log: [binance_api_rate_limit_test_request_threaded#1.log](log/binance_api_rate_limit_test_request_threaded%231.log) 
* Interval between requests: 0ms
* Time taken to send 1200 requests and receive responses: 2.8s

The following error occured attempting to send almost 1200 requests without an interval.
```
APIError(code=-1003): Too many requests; current limit is 1200 requests per minute.
Please use the websocket for live updates to avoid polling the API.
```

The following error occured continuously attempting to send more requests.
```
APIError(code=-1003): Way too many requests; IP banned until 1535856370598.
Please use the websocket for live updates to avoid bans.
```

### Threaded #2 with intervals between requests

* Log: [binance_api_rate_limit_test_request_threaded#2.log](log/binance_api_rate_limit_test_request_threaded%232.log) 
* Interval between requests: 40ms
* Time taken to send 1200 requests and receive responses: 52.1s

The following error occured attempting to send almost 1200 requests at 40ms interval.
```
APIError(code=-1003): Too many requests; current limit is 1200 requests per minute.
Please use the websocket for live updates to avoid polling the API.
```

The following error occured continuously attempting to send more requests.
```
APIError(code=0): Invalid JSON error message from Binance: <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML><HEAD><META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=iso-8859-1">
<TITLE>ERROR: The request could not be satisfied</TITLE>
</HEAD><BODY>
<H1>403 ERROR</H1>
<H2>The request could not be satisfied.</H2>
<HR noshade size="1px">
Request blocked.

<BR clear="all">
<HR noshade size="1px">
<PRE>
Generated by cloudfront (CloudFront)
Request ID: 8ofuHqdTfukCBfuM0BWYcAWYO0sNRe4EV3oEZPIgNeSitIteMVFlSQ==
</PRE>
<ADDRESS>
</ADDRESS>
</BODY></HTML>
```

### Conclusion

It is confirmed that the rate limit of requests is 1200 per minute.
So the safe number of requests per second is 20(=1200/60) on average.

## Order - New order  (TRADE)

```
POST /api/v3/order  (HMAC SHA256)
```
* **Weight:** 1

### Non-threaded #1

* Log: [binance_api_rate_limit_test_order_non-threaded#1.log](log/binance_api_rate_limit_test_order_non-threaded%231.log)
* Interval between requests: 0ms
* Time taken to send 10 orders and receive responses: 0.5s
 
The following error occured attempting to send 11 orders.
```
APIError(code=-1015): Too many new orders; current limit is 10 orders per SECOND.
```

### Non-threaded #2

* Log: [binance_api_rate_limit_test_order_non-threaded#2.log](log/binance_api_rate_limit_test_order_non-threaded%232.log)
* Interval between requests: 0ms
* Time taken to send 16 orders and receive responses: 1.9s
 
Error didn't occur because the first 7 orders were sent from 09:40:**33**.614 to 09:40:**33**.960 and the following 9 orders were sent from 09:40:**34**.015 to 09:40:**34**.476.

### Conclusion

It is confirmed that the rate limit of orders is 10 per second.
It might be correct that one day limit is 100000 according to official API.
So the safe number of orders per second during the day is 1.15(=100000/86400) on average.
