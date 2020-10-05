# Getting started

> This guide is intended to be used for integration with Skyee’s Accommodation/Tuition collection service via Bank Transfer.

## Quick start

If you intend to integrate Skyee's collection service into your website,you will need to do the following steps:

* Acquire a merchantId from Skyee's account manager.
* Acquire an appId from Skyee's account manager.
* Get a certificate key from Skyee so you can verify the message signature sent by Skyee.
* Issue a certificate key to Skyee so Skyee can verify the message signature

1. You use all the information that you are given to create a link, where your payer will click and be brought to a Skyee hosted web page.

    Link format is like the following:
```
    https://pay.skyeepayment.com/web/pay?merchantId=123&appId=984342&orderId=12345-12345767-12345677&orderTime=15583942134&c1=GBP&c2=CNY&amt1=1023111&amt2=9484213&productId=10000&notificationUrl=https://merchant.com&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI
```
2. Within the Skyee hosted web page, Skyee will display the following information to the payer:

    * OrderId from the merchant
    * Payment amount in its original currency
    * Payment amount to be collected in CNY format
    * Time the order was created


> The payer will be asked to provide all necessary information on the page and submit all files.

3. Once everything submitted, Skyee will display the account detail which can serve as the payee account for the payer transfer fund to. As this is an offline bank transfer, Skyee will also generate and display a PIN code which the payer MUST put together with the transfer in the comment or note field, for Skyee to identify the payment. A Unique Pin code will be associated with an order from the merchant and the PIN code will expire in 48 hours, which should be enough for the payer to complete the bank transfer.

### Parameter Details

```
https://pay.skyeepayment.com/web/pay?merchantId=123&appId=984342&orderId=12345-12345767-12345677&orderTime=15583942134&c1=GBP&c2=CNY&amt1=1023111&amt2=9484213&productId=10000&notificationUrl=https://merchant.com&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI

```

Parameter | Type | Description 
--------- | ------- | ----------- |
merchantId | string |  `required`<br> a unique ID given by Skyee to identify the merchant.
appId | string | `required`<br> each accommodation company’s ID, issued after KYCC done.
orderId | string | `required` `Maxlength: 32`<br>  the merchant’s order number. It must be unique for the merchant. Alphanumeric, "_" and "-" are allowed.  
payType | string | `mandatory`<br>  Default is `BankTransfer`.
orderTime | integer | `required`<br> when is this order created. Unix Timestamp, e.g. 15583942134.
productId | integer | `required`<br> the accommodation's productId is 10000
c1 | string | `required`<br> Original currency in GBP/USD/AUD/NZD/EUR. 
amt1 | integer | `required`<br> Original amount to display to the student.<br>The original amount will use the smallest unit of currency, but the reference value must be an integer without decimals. For example, using a currency type of "United States dollars" a reference value of "1750" in the payment amount would be equivalent to US$17.50.
c2 | string | `Conditional`<br>  Collection currency which will be CNY.
amt2 | integer | `Conditional`<br> CNY collection amount which Skyee will collect from the student. <br><br> According to the agreement with Skyee, if you are to instruct how much CNY that Skyee must collect from the student, the c2 & amt2 fields are mandatory.<br><br> If within the agreement that you only need to specify the c1 & amt1, the c2 & amt2 fields will be auto dropped even if there are values in the two fields.<br><br>E.g. a student must pay USD 10,000 to a merchant. The merchant has an agreement with Skyee that the merchant will receive exactly USD 10,000 from the student. Then Skyee will use a real time FX rate to calculate how much CNY this student must pay.<br><br>But if the merchant instruct Skyee to collect CNY 70,000 from the student, then Skyee will have no way to guarantee that the merchant will eventually receive USD 10,000 due to a constantly changing FX rate.<br><br>**The collection amount will use the smallest unit of CNY, but the reference value must be an integer without decimals. For example, using a currency type of "China Yuan" a reference value of "1750" in the payment amount would be equivalent to CNY￥17.50.**
notificationUrl | string | `required` <br>The URL where we will push async notifications, so the merchant could receive payment result if it's generated a bit later.
sign | string |  `required` <br>Signature encrypted using RSA(SHA1WithRSA) signature. Reference  [Create a signature](#create-a-signature)

### Notification

Skyee needs a URL endpoint which Skyee could send payment notification to. It’s a POST request to inform the merchant of the payment status, a code sample would be like following:

```Shell

POST "notificationUrl" HTTP/1.1 Content-Type: application/x-www-form-urlencoded

merchantId=8731769123&appId=758353&payType=offlineTransfer&orderId=12345-12345767-12345677&orderTime=15583942134&productId=10000&orderStatus=paid&c1=GBP&c2=CNY&amt1=1023111&amt2=9484213&errorCode=xxx&sign=URgwbxHfL%2FE3YNiIBpP0vbL1UPtvbsqfAvGMpLFo5nIW2Bq786Mi0uLrvsI

```

In the POST, the following parameters will be sent:

#### Parameters

Parameter | Type | Description
--------- | ------- | ----------- |
merchantId | string |  `readonly`<br> a unique ID given by Skyee to identify the merchant.
appId | string | `readonly`<br> each accommodation company’s ID.
orderId | string | `readonly` `Maxlength: 32`<br>  the merchant’s order number.
payType | string | `readonly`<br> Payment Method
orderTime | integer | `readonly`<br> when is this order created. Unix Timestamp, e.g. 15583942134.
productId | integer | `readonly`<br> the accommodation's productId is 10000
c1 | string | `readonly`<br> Original currency in GBP/USD/AUD/NZD/EUR. 
amt1 | integer | `readonly`<br> Original amount to display to the student.<br> The original amount will use the smallest unit of currency, but the reference value must be an integer without decimals. For example, using a currency type of "United States dollars" a reference value of "1750" in the payment amount would be equivalent to US$17.50.
c2 | string | `readonly`<br>  Collection currency which will be CNY.
amt2 | integer | `readonly`<br> CNY collection amount which Skyee will collect from the student.<br> The collection amount will use the smallest unit of CNY, but the reference value must be an integer without decimals. For example, using a currency type of "China Yuan" a reference value of "1750" in the payment amount would be equivalent to CNY￥17.50.
orderStatus | string |  `readonly`<br> Order Status. 
errorCode  | string |  `readonly`<br> The error code when an error occurs.
sign | string | Signature encrypted using RSA(SHA1WithRSA) signature. Reference  [verify a signature](#verify-a-signature)

### Create a signature

1. Presume all data sent  is the set M. Sort non-empty values in M in ascending alphabetical order (i.e. lexicographical sequence), and join them into string A via the corresponding URL key-value format (e.g. key1=value1&key2=value2…).

    Notes:

    * Sort parameter names in ascending alphabetical order based on their ASCII encoded names (e.g. lexicographical sequence)
    * Empty parameter values are excluded in the signature;
    * Parameter names are case-sensitive;
    * When checking returned data or a Skyee push notification signature, the transferred sign parameter is excluded in this signature as it is compared with the created signature.

2. Perform SHA1 arithmetic on stringA,  thus get sign's value (signValue).

    Example:

    For the following transferred parameters:

    merchantId：1000000000 <br>
    appId：10000100 <br>
    orderId： 12345-12345767-12345677 <br>
    orderTime：15583942134 <br>
    c1：GBP <br>
    amt1：1023111 <br>
    c2: CNY <br>
    amt2: 9484213 <br>
    productId: 10000 <br>
    notificationUrl: https://merchant.com

    a. Sort ASCII code of parameter names by lexicographical sequence based on the format of "key=value"
    ```
    stringA="amt1=1023111&amt2=9484213&appId=10000100&c1=GBP&c2=CNY&merchantId=1000000000&notificationUrl=https://merchant.com&orderId=12345-12345767-12345677&orderTime=15583942134&productId=10000";
    ```
    b. The merchant sign the signature string using the merchant's RSA private key

```C#
    // C# Sample Code
    var dic = new Dictionary<string, string>()
    {
        { "merchantId", "1000000000" },
        { "appId", "10000100"},
        { "orderId","12345-12345767-12345677"},
        { "c1","GBP"},
        { "amt1","1023111"},
        { "c2","CNY"},
        { "amt2","9484213"},
        { "orderTime", "15583942134"},
        { "productId", "10000"},
        { "notificationUrl" , "https://merchant.com"},
    };

    Dictionary<string, string> ascDic = param.OrderBy(o => o.Key).ToDictionary(o => o.Key, p => p.Value);
    var sb = new StringBuilder();

    foreach (var item in ascDic)
    {
        if (!string.IsNullOrEmpty(item.Value))
        {
            sb.Append(item.Key).Append("=").Append(item.Value).Append("&");
        }
    }

    string stringA = sb.ToString().Substring(0, sb.ToString().Length - 1);

    var certPrivateKey = File.ReadAllText("merchant-private-key.pem");
    certPrivateKey = certPrivateKey.Replace("-----BEGIN RSA PRIVATE KEY-----", "")
                                   .Replace("-----END RSA PRIVATE KEY-----", "")
                                   .Replace("-----BEGIN RSA PUBLIC KEY-----", "")
                                   .Replace("-----END RSA PUBLIC KEY-----", "")
                                   .Replace("-----BEGIN PRIVATE KEY-----", "")
                                   .Replace("-----END PRIVATE KEY-----", "")
                                   .Replace("-----BEGIN PUBLIC KEY-----", "")
                                   .Replace("-----END PUBLIC KEY-----", "")
                                   .Replace(Environment.NewLine, "");
    var rsa = RSA.Create();
    int bytesRead;
    rsa.ImportPkcs8PrivateKey((ReadOnlySpan<byte>)Convert.FromBase64String(certPrivateKey), out bytesRead);
    var signData = rsa.SignData(Encoding.UTF8.GetBytes(stringA), HashAlgorithmName.SHA1, RSASignaturePadding.Pkcs1);
    var sign = Convert.ToBase64String(signData);
```

### Verify a signature

1. Presume all data received(excluded sign field) is the set M. Sort non-empty values in M in ascending alphabetical order (i.e. lexicographical sequence), and join them into string A via the corresponding URL key-value format (e.g. key1=value1&key2=value2…).

    Notes:

    * Sort parameter names in ascending alphabetical order based on their ASCII encoded names (e.g. lexicographical sequence);

    * Empty parameter values are excluded in the signature;

    * Parameter names are case-sensitive;

    * When Skyee push notification signature, the transferred sign parameter is excluded in this signature as it is compared with the created signature.

2. Perform SHA1 arithmetic on stringA,  thus get sign's value (signValue).

    Example:

    For the following transferred parameters:

    merchantId：1000000000 <br>
    appId：10000100 <br>
    orderId： 12345-12345767-12345677 <br>
    payType: BankTransfer <br>
    orderTime：15583942134 <br>
    c1：GBP <br>
    amt1：1023111 <br>
    c2: CNY <br>
    amt2: 9484213 <br>
    productId: 10000 <br>
    orderStatus: Paid <br>
    errorCode : <br>
    sign: CzSlJ6X4nPsP5p36PQYf51a9OJRK6+V65kFwuD4wxeRdeoJZensPCD0YBB3uUlSphbR1RXACfSMXy9nfanbmh8n+dk8mUJiN2G4+Xwd0/HuYQqv9jUMMJT4UkfWO9CARlDrF4dUHDV4U7d4dcb4HjbRFeHyPbK2utWlHGQ1HsjJ6nupW2kxWIZ6MOVu0V5E+OUO1A5qMbjvIm8Etx5nEW/3v+qkH19S66FwjP0bh9HfYBGmA64YZHbc4W+3R5OF16EvGOm8sXNyF+9+IxUmbWrJ/a5GP6y8wvI6b7TZorjD1ss41Xuvuow0dtdwbXZn8ZPHcyxz2bdssh5ha9UEP7g==

    a. Sort ASCII code of parameter names by lexicographical sequence based on the format of "key=value"
    ```
    stringA="amt1=1023111&amt2=9484213&appId=10000100&c1=GBP&c2=CNY&merchantId=1000000000&orderId=12345-12345767-12345677&orderStatus=Paid&orderTime=15583942134&payType=BankTransfer&productId=10000";
    ```
    b. The merchant verify the signature string using the Skyee's RSA public key
    
    Environment | Public key 
    --------- | ------- 
    Sandbox | -
    Production | [Download Skyee Public Key](https://pay.skyeepayment.com/files/publickey)

```
    // C# Sample Code
using RSAExtensions;

    var dic = new Dictionary<string, string>()
    {
        { "merchantId", "1000000000" },
        { "appId", "10000100"},
        { "orderId","12345-12345767-12345677"},
        { "payType" , "BankTransfer"},
        { "c1","GBP"},
        { "amt1","1023111"},
        { "c2","CNY"},
        { "amt2","9484213"},
        { "orderTime", "15583942134"},
        { "productId", "10000"},
        { "orderStatus" , "Paid"},
        { "errorCode" , ""},
    };

    string sign =
        "CzSlJ6X4nPsP5p36PQYf51a9OJRK6+V65kFwuD4wxeRdeoJZensPCD0YBB3uUlSphbR1RXACfSMXy9nfanbmh8n+dk8mUJiN2G4+Xwd0/HuYQqv9jUMMJT4UkfWO9CARlDrF4dUHDV4U7d4dcb4HjbRFeHyPbK2utWlHGQ1HsjJ6nupW2kxWIZ6MOVu0V5E+OUO1A5qMbjvIm8Etx5nEW/3v+qkH19S66FwjP0bh9HfYBGmA64YZHbc4W+3R5OF16EvGOm8sXNyF+9+IxUmbWrJ/a5GP6y8wvI6b7TZorjD1ss41Xuvuow0dtdwbXZn8ZPHcyxz2bdssh5ha9UEP7g==";
    byte[] signature = Convert.FromBase64String(sign);

    Dictionary<string, string> ascDic = dic.OrderBy(o => o.Key).ToDictionary(o => o.Key, p => p.Value);
    var sb = new StringBuilder();

    foreach (var item in ascDic)
    {
        if (!string.IsNullOrEmpty(item.Value))
        {
            sb.Append(item.Key).Append("=").Append(item.Value).Append("&");
        }
    }

    var stringA = sb.ToString().Substring(0, sb.ToString().Length - 1);
    var certPublicKey = File.ReadAllText("skyee-publick-key.pem");
    var rsa = RSA.Create();
    rsa.ImportPublicKey(RSAKeyType.Pkcs8, certPublicKey, isPem: true);
    bool isOK = rsa.VerifyData(Encoding.UTF8.GetBytes(stringA), signature, HashAlgorithmName.SHA1, RSASignaturePadding.Pkcs1); //verify sign
    Assert.True(isOK);
```