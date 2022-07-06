### はじめに
`Easy transfer Qr Code`機能はiOS版LINE 12.10.0から実装された機能です。

### 解析
1.　以下の構造体はEasy transfer Qr Codeで生成されます。
SessionIdは`createSession()`で生成します。

```http
POST /EXT/auth/feature-user/api/primary/mig/qr/prepare HTTP/2
Host: gw.line.naver.jp
Content-Type: application/x-thrift
X-Line-Application: IOS	12.10.0	iOS	15.5
X-Line-Access: "V3Token"
Accept: application/x-thrift
X-Lhm: POST
X-Lpv: 1
User-Agent: Line/12.10.0 iPhone13,3 15.5
X-Lal: en_JP

createSession
```

QRコード内のデータは、以下のようになっています。
```go
type TransferQrCode struct {
	QrCodeIdentifier string `json:"qi"`
	PublicKey        string `json:"pk"`
	SessionId        string `json:"si"`
}
```
2. QRCodeでアカウントを引き継ぐ

```
sk, pk = generateCurve25519Key()
sharedSecret = ECDH(sk, PublicKey)

plaintext = hex.DecodeString(QrCodeIdentifier)
nonce = rng(12)
encryptedQrIdentifier = AESGCMSIV(key, nonce, plaintext)
encryptedQrIdentifier = append(nonce, encryptedQrIdentifier...)

res, err := CheckIfEncryptedE2EEKeyReceived(ctx, &CheckIfEncryptedE2EEKeyReceivedRequest{
    SessionId: SessionId,
	SecureChannelData: &SecureChannelData{
		NewDevicePublicKey:   pk,
		EncryptedQrIdentifier: fmt.Sprintf("%X", encryptedQrIdentifier),
		},
})
```

`X-Line-Access`はSessionIdです

```http
POST /EXT/auth/feature-user/lp/api/primary/mig/qr HTTP/2
Host: gw.line.naver.jp
X-Lal: ja_JP
X-Line-Access: "SessionId"
X-Line-Application: IOS	12.10.0	iPadOS	15.5
User-Agent: Line/12.10.0 iPad5,1 15.5
Content-Type: application/x-thrift
Accept: application/x-thrift
X-Lpv: 1
X-Lhm: POST

checkIfEncryptedE2EEKeyReceived
```

ToDo: `EncryptedSecureChannelPayload`の解析

実際のデータ
![line-12-10-0](https://user-images.githubusercontent.com/89930816/176069331-7e8bff05-971b-4e35-b554-7b4e0550b755.jpg)
