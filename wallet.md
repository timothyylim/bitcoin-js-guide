### HD Wallets
* Deterministic wallets based on BIP-32
* To backup them up, all you need is the seed Mnemonic

### Mnemonic
* Standard defind in BIP-39
* The final mnemonic is 12-24 words
* Not completely random words since the last word is a checksum  
    ```split logic consider degree smile field term style opera dad believe indoor item type beyond```  
* Pg 101, Fig 5-6: Entropy to Mnemonic

### 512-bit Root Seed
* Represented in Hex Format (Each hex digit stores 4 bits)
    ```f8355cdca603c05231502f26e8f36134fd6723a90aa544160322be69236912f93e14d015ca448b1fcafe2685db2a571c3006c3b280d6885a3fd3c9ec3e9b62d7```
* Length of the seed is 128 which presents 128*4 = 512 bits
* Pg 102, Fig 5-7: Mnemonic to Seed

### Hashing Root Seed
* 512-bit seed is input into an HMAC-SHA512 hash function to generate a 512 bit number
* Pg 106, Fig 5-9: Master keys and Chain Code from Root seed
* **Master Private Key:** left 256 bits of the hash output
* **Master Chain Key:** derived from the public key (264 bits)
* **Master Chain Code:** right 256 bits of hash output
* The master/parent private key, the chain code, and an index number (32 bits) can then be used to create child keys and chain codes
* Page 107, Fig 5-10

### Extended Keys
* **Extended Private Key**: 
  * 512 bits: Private Key and chain code
  * Encoded using Base58Check, and start with the prefix xprv
  ```xprv9uMQ2mvE3MkzhAt2DbK8tKXbVVw4DB3LjjNDXSNCn6QLC9Gxg9vqD9oj92nP2tn4pwiP5ALPFWWnMytYTVdtYZPhukvraoY9EVpsmDMcXJn ```
* **Extended Public Key**: 
  * 512 bits: Pulic Key and chain code
  * Encoded using Base58Check, and start with the prefix xpub
  ```xpub68LkSHT7sjKHuexVKcr9FTUL3XmYcdmC6xHpKpmpLRwK4wc7DhF5kx8CzH9DumfEuTggLTbeGCE6e5YB3F4JL3NaM8jtffXBHE89jySaCX9 ```

### Base58CheckEncoding
* Base64: 26 lowercase characters, 26 uppercase characters, 10 numerals, +, / (64 in total)
* Base58: Base64 without 0,O,l,I,+,/ (58 characters)
* Base58CheckEncoding = Prefix + data + Checksum
  * **Prefix:**
    * 0x80 for a Private Key WIF (5,K or L in Base58)
    * 0x00 for a Bitcoin address (1 in Base58)
    * 0x6F for TestNet address (m or n in Base 58)
    * 0x0488B21E for Extented Public Key (xpub in Base 58)
    * Pg 68, Tb 1
  * **Checksum**:
    * SHA256(SHA256(prefix(prefix+data)))
    * First 4 bytes of the above mentioned 32 byte number
 
 ### Private Key
 * WIF encoding 
    * Base58CheckEncoding with 0x80 prefix, and 32 bitcheck sum as desribed above
    * Starts with 5 in in Base58 format
 * WIF-compressed
    * Same as WIF with 0x01 appended to the data before the encoding
    * Starts with K or L
    * The format used by https://iancoleman.io/bip39/ 
 * The raw key in hex, without the Base58CheckEncoding or WIF-compressed suffix (0x01) is 256 bits (64 hex digits)
 * From WIF compressed to hex:
    ```javascript
    const bs58 = require('bs58')
    const assert = require('assert')

    const key = 'KzPF7MF5U5DTfGidiy2MgvZf6k8HJYxvQ3YzMUUbS6UjKmd1PRfc'
    // WIF compressed because it starts with a K
    const bytes = bs58.decode(key)
    const hexKey = bytes.toString('hex')
    console.log(hexKey)
    // 805e6a9b2782ec3fcac4d91cf796f9888c2deea6ea16bb68853e9b292f2978227b01f0eb020f
    console.log(hexKey.length)
    // 76
    console.log('Prefix:' + hexKey.slice(0, 2))
    // Prefix:80
    const data = hexKey.slice(2, hexKey.length - 10)
    console.log('Data:' + data)
    // Data:5e6a9b2782ec3fcac4d91cf796f9888c2deea6ea16bb68853e9b292f2978227b
    assert.strictEqual(data.length, 64)
    console.log('WIF-compressed:' + hexKey.slice(hexKey.length - 10, hexKey.length - 8))
    // 01 at the end inidicates that this is WIF compressed, not just WIF
    console.log('Suffix:' + hexKey.slice(hexKey.length - 8, hexKey.length))
    // Suffix:f0eb020f
    ```
    
### Public Key
* We're referring to compressed public keys here (Pg 74, Fig 4-7)
* 264 bits (66 hex digits), 8 bits more than the private key
* Obtained from the priv Key using Eliptical Curve Cryptography
* Getting private and public key from WIF Compressed Private Key
    ```javascript
    const bitcoin = require('bitcoinjs-lib')
    const assert = require('assert')

    const refPrivKey = 'KzPF7MF5U5DTfGidiy2MgvZf6k8HJYxvQ3YzMUUbS6UjKmd1PRfc' // WIF Compressed
    const refPubKey = '03107f1be6a20e51f52f8757b0976cb9f695306cd1f80bb4a9fcd59e54c9a77cc1' // Hex
    const refAddress = '1CxYe5agV1mvCJiRPSBkNRnS9uNRbQDLzw' // Base58CheckEncoding
    // As obtained by https://iancoleman.io/bip39/
    const data = '5e6a9b2782ec3fcac4d91cf796f9888c2deea6ea16bb68853e9b292f2978227b'
    // As obtained in the WIF compressed to hex conversion above
    const keyPair = bitcoin.ECPair.fromWIF(refPrivKey)
    const pubKey = keyPair.publicKey.toString('hex')
    const privKey = keyPair.privateKey.toString('hex')
    const { address } = bitcoin.payments.p2pkh({ pubkey: keyPair.publicKey })

    assert.strictEqual(pubKey, refPubKey)
    assert.strictEqual(pubKey.length, 66)
    assert.strictEqual(privKey, data)
    assert.strictEqual(address, refAddress)
    ```

### Address
* Base58CheckEncoding with 0x00 prefix, and 32 bitcheck sum as desribed above
* Similar to Private Key, without the 0xb01 suffix for WIF-compressed, and different 8 bit prefix
* 160 bits, or 40 hex digits
* Pg 66, Fig 4-5: Public Key to Bitcoin Address
* Base58CheckEncoding to hex
    ```javascript
    const bs58 = require('bs58')
    const assert = require('assert')

    const address = '1CxYe5agV1mvCJiRPSBkNRnS9uNRbQDLzw'
    // WIF compressed because it starts with a 1
    const bytes = bs58.decode(address)
    const hexAddress = bytes.toString('hex')
    console.log(hexAddress)
    // 00832aac3432848624184dc806e8a229c41e560e4041d6eedc
    console.log(hexAddress.length)
    // 50
    console.log('Prefix:' + hexAddress.slice(0, 2))
    // Prefix:00
    const data = hexAddress.slice(2, hexAddress.length - 8)
    console.log('Data:' + data)
    // Data:832aac3432848624184dc806e8a229c41e560e40
    assert.strictEqual(data.length, 40)
    console.log('Suffix:' + hexAddress.slice(hexAddress.length - 8, hexAddress.length))
    // Suffix:41d6eedc
    ```

### Links
* https://iancoleman.io/bip39/ 
* https://github.com/cryptocoinjs/bs58
