
#   Skyee Accommodation / Tuition Integration Guide (Offline Bank Transfer) 

>Note: This guide is intended to be used for integration with Skyee’s Accommodation/Tuition collection service via OfflineBank Transfer.
>
If you intend to integrate Skyee's collection service into your website, you will need to do the following steps:

    1. Acquire a merchantId from Skyee's account manager.
    2. Acquire an appId from Skyee's account manager.
    3. Get a certificate key from Skyee so you can verify the message signature sent by Skyee.
    4. Issue a certificate key to Skyee so Skyee can verify the message signature sent with your request.

With the above information ready, a general payment process would be like:

   1. You use all the information that you are given to create a link, where your payer will click and be brought to a Skyee hosted web page. Link format is like the following:

``` 
https://pay.skyeepayment.com/pay/offlinetransfer?merchantId=123&appid=984342&orderId=12345-12345767-12345677&orderTime=20200901020101&c1=GBP&c2=CNY&amt1=1023111&amt2=9484213&notificationUrl=https://merchant.com&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI
```

2. Within the Skyee hosted web page, Skyee will display the following information to the payer:

* OrderId from the merchant
* Payment amount in its original currency
* Payment amount to be collected in CNY format
* Time the order was created  
    
    **The payer will be asked to provide all necessary information on the page and submit all files.**
    
3. Once everything submitted, Skyee will display the account detail which can serve as the payee account for the payer transfer fund to. As this is an offline bank transfer, Skyee will also generate and display a PIN code which the payer MUST put together with the transfer in the comment or note field, for Skyee to identify the payment. A Unique Pin code will be associated with an order from the merchant and the PIN code will expire in 48 hours, which should be enough for the payer to complete the bank transfer.

##  Parameter details:
```
https://pay.skyeepayment.com/pay/offlinetransfer?merchantId=123&appId=984342&orderId=12345-12345767-12345677&orderTime=20200901020101&c1=GBP&c2=CNY&amt1=1023111&amt2=9484213&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI
```
* merchantId: a unique ID given by Skyee to identify the merchant
* appid: each accommodation company’s ID, issued after KYCC done
* orderId: the merchant’s order number
* orderTime: when is this order created,For example 20171117020101
* c1: Original currency in GBP/USD/AUD/NZD/EUR
* c2: Collection currency which will be CNY
* amt1: Original amount to display to the student  
* amt2: CNY collection amount which Skyee will collect from the student  
* notificationUrl: The URL where we will push async notifications, so the merchant could receive payment result if it's generated a bit later. 
* sign: Signature encrypted using RSA(SHA1WithRSA) signature。
```
amt1=1023111&amt2=9484213&appid=984342&c1=GBP&c2=CNY&merchantId=123&orderId=12345-12345767-12345677&orderTime=20200901020101
```

RSA(SHA1WithRSA) signature logic:
1. The signature string consists of key-value pairs for non-empty fields other than sign.
2. All fields participating in the signature, sorted by the Ascii code of the field name, are concatenated into a string using the key-value pair format of the URL (i.e., key1=value1& Key2 =value2).
3. Sign with the merchant's RSA private key.sign=rsa(string.getbyte("utf-8"),cusRsaPrivateKey);

## Notification:
Skyee needs a URL endpoint which Skyee could send payment notification to. It’s a POST request to inform the merchant of the payment status, a code sample would be like following:

```
POST "notificationUrl" HTTP/1.1 Content-Type: application/x-www-form-urlencoded

merchantId=8731769123&appId=758353&payType=offlineTransfer&orderId=12345-12345767-12345677&orderTime=20200901020101&orderStatus=paid&errorCode=xxx&errorMsg=xxx&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI    
```
In the POST, the following parameters will be sent:

* merchantId
* appId
* payType
* orderId
* orderTime
* orderStatus
* errorCode
* errorMsg
* sign

For order status  

* paid: the amount of CNY has been well received.
* paymentFailed: within 48 hours the payment is not received, please be noted the notifiction itself may be sent after 48 hours timeframe.
* refund: for some reason Skyee has to make a refund to the student
* paymentSent: the payment has been sent to the merchant


