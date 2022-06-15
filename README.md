# PeopleWallet
A minimal javascript wallet for Node and browser.

## About
Peoplewallet is a HD wallet that can store your private keys encrypted in the browser to allow you to run Ethereum dapps even if you're not running a local Ethereum node. It uses BIP32 and BIP39 to generate an HD tree of addresses from a randomly generated 12-word seed.

Peoplewallet is primarily intended to be a signing provider for the Hooked Web3 provider through the keystore module. This allows you to have full control over your private keys while still connecting to a remote node to relay signed transactions. Moreover, the txutils functions can be used to construct transactions when offline, for use in e.g. air-gapped coldwallet implementations.

The default BIP32 HD derivation path has been m/0'/0'/0'/i, but any HD path can be chosen.

## Security
Please note that Peoplewallet has not been through a comprehensive security review at this point. It is still experimental software, intended for small amounts of Ether to be used for interacting with smart contracts on the Ethereum blockchain. Do not rely on it to store larger amounts of Ether yet.

## Get Started
```bash
npm install peoplewallet
```

The Peoplewallet package contains dist/Peoplewallet.min.js that can be included in an HTML page:

```html
<html>
  <body>
    <script src="Peoplewallet.min.js"></script>
  </body>
</html>
```

The file Peoplewallet.min.js exposes the global object Peoplewallet to the browser which has the two main modules Peoplewallet.keystore and Peoplewallet.txutils.

Sample recommended usage with hooked web3 provider:

```js
// the seed is stored encrypted by a user-defined password
var password = prompt('Enter password for encryption', 'password');

keyStore.createVault({
  password: password,
  // seedPhrase: seedPhrase, // Optionally provide a 12-word seed phrase
  // salt: fixture.salt,     // Optionally provide a salt.
                             // A unique salt will be generated otherwise.
  // hdPathString: hdPath    // Optional custom HD Path String
}, function (err, ks) {

  // Some methods will require providing the `pwDerivedKey`,
  // Allowing you to only decrypt private keys on an as-needed basis.
  // You can generate that value with this convenient method:
  ks.keyFromPassword(password, function (err, pwDerivedKey) {
    if (err) throw err;

    // generate five new address/private key pairs
    // the corresponding private keys are also encrypted
    ks.generateNewAddress(pwDerivedKey, 5);
    var addr = ks.getAddresses();

    ks.passwordProvider = function (callback) {
      var pw = prompt("Please enter password", "Password");
      callback(null, pw);
    };

    // Now set ks as transaction_signer in the hooked web3 provider
    // and you can start using web3 using the keys/addresses in ks!
  });
});
```
Sample old-style usage with hooked web3 provider (still works, but less secure because uses fixed salts).

```js
// generate a new BIP32 12-word seed
var secretSeed = Peoplewallet.keystore.generateRandomSeed();

// the seed is stored encrypted by a user-defined password
var password = prompt('Enter password for encryption', 'password');
Peoplewallet.keystore.deriveKeyFromPassword(password, function (err, pwDerivedKey) {

var ks = new Peoplewallet.keystore(secretSeed, pwDerivedKey);

// generate five new address/private key pairs
// the corresponding private keys are also encrypted
ks.generateNewAddress(pwDerivedKey, 5);
var addr = ks.getAddresses();

// Create a custom passwordProvider to prompt the user to enter their
// password whenever the hooked web3 provider issues a sendTransaction
// call.
ks.passwordProvider = function (callback) {
  var pw = prompt("Please enter password", "Password");
  callback(null, pw);
};

// Now set ks as transaction_signer in the hooked web3 provider
// and you can start using web3 using the keys/addresses in ks!
});
```

## License
MIT License.