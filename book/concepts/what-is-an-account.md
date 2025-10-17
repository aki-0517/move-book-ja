# アカウント（Account）

<!--

- user is an account
    - account is identified by an address
    - account is generated from a private key
    - account can own objects
    - account can send transactions
    - every transaction has a sender
    - sender is identified by an address
    - sui cryptographic agility
    - sui account types
    - supported curves: ed25519, secp256k1, zklogin

 -->

アカウントはユーザーを識別する方法です。アカウントは秘密鍵から生成され、
アドレスによって識別されます。アカウントはオブジェクトを所有でき、トランザクションを送信できます。すべてのトランザクション
には送信者があり、送信者は[アドレス](./address)によって識別されます。

Suiはアカウント生成のために複数の暗号化アルゴリズムをサポートしています。サポートされている2つの曲線は
ed25519とsecp256k1で、またアカウントを生成する特別な方法であるzkloginもあります。
暗号化アジリティ（Suiの独自機能）により、アカウント生成の柔軟性が可能になります。

<!-- The cryptographic agility allows for flexibility in the account generation -->

## 関連資料

- [Sui Blog](https://blog.sui.io)の[Cryptography in Sui](https://blog.sui.io/wallet-cryptography-specifications/)
- [Sui Docs](https://docs.sui.io)の[Keys and Addresses](https://docs.sui.io/concepts/cryptography/transaction-auth/keys-addresses)
- [Sui Docs](https://docs.sui.io)の[Signatures](https://docs.sui.io/concepts/cryptography/transaction-auth/signatures)
