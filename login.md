### 1. Introduction
This content will share the results of our analysis of how we reverse engineered the LINE application.    
**Please note** that the analysis results are as of LINE version (11.18.0), so the specifications may change in future versions.

### 2. Communication protocol analysis
In this content, we used the Burp suite to analyze the packets.    
As a result of the analysis, thrift[^1] was used as the communication protocol.
[^1]: https://thrift.apache.org/

> The login session will use the smartphone as a key (Primary As A Key)
![Screen Shot01](https://user-images.githubusercontent.com/89930816/138833004-044786c2-6720-458a-b5e2-6f0ec739e2fe.png)

> Generate a Curve25519 key pair and share it with the primary device.    
> `temporalPublicKey` is the base64-encoded public key of the generated key pair.

`e2eeVersion` is always 1 (may change in the future).
```go
sk, pk := curve25519KeyPair()


exchangeKey := map[string]string{
    "temporalPublicKey": base64(pk),
    "e2eeVersion": "1"
}
```

> If the return value is an empty structure, it is the same as void.
![Screen Shot02](https://user-images.githubusercontent.com/89930816/138833425-77af4dde-afea-4d03-a236-8fefcf0b2c8a.png)

### 3. Thrift

Explain the request in more detail.    
Message　header: `protocolID`, `ver_type`, `sequenceId`(First 3Bytes), `name_length`, `name`    
WriteMessageBegin(protocolID(0x082), ver_type(21), sequenceId(0),  name_length(0e) name(70 75 74 45 78 63 68 61 6E 67 65 4B 65 79))    
<img width="616" alt="Screen Shot03" src="https://user-images.githubusercontent.com/89930816/138833534-14d5dc2a-88c8-4853-b5d4-fc538e6350fa.png">

<img width="561" alt="Screen Shot 04" src="https://user-images.githubusercontent.com/89930816/138837573-9a28fd4a-d7cd-4306-9eda-97396f52850e.png">

<img width="675" alt="Screen Shot 2021-10-26 at 16 49 21" src="https://user-images.githubusercontent.com/89930816/138843095-41c826e9-2ec3-4df3-b6aa-643371f14f60.png">
