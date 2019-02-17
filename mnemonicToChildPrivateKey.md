# How to create a child private key from a Mnemonic 

```javascript
const bip32 = require('bip32')
const bip39 = require('bip39')

const mnemonic = 'split logic consider degree smile field term style opera dad believe indoor item type beyond'
const seed = bip39.mnemonicToSeed(mnemonic)
console.log(seed.toString('hex'))
// f8355cdca603c05231502f26e8f36134fd6723a90aa544160322be69236912f93e14d015ca448b1fcafe2685db2a571c3006c3b280d6885a3fd3c9ec3e9b62d7
const path = 'm/0/0'
const node = bip32.fromSeed(seed)
const child1 = node.derivePath(path)
console.log(child1.toWIF())
// L1T5525SbSDjM1DzMfLXx9CSjHVCicVH5cectxEbrwxQdvqxEwL8
```
