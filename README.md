# 📘 Bitcoin CLI & Wallet Tutorial — Setup, Watch-Only Wallets, Full Wallets, and Electrum Integration Xpub Export

## 📌 Versioning & Dependency Setup

```bash
git tag -n | sort -V
git checkout v29.0
sudo apt-get install bison -y
sudo apt-get install byacc -y
```

## ⚙️ Building Bitcoin Core

```bash
cd depends
make -j$(nproc)
```

```bash
bitcoind -conf=/home/<your-user>/projects/bitcoin/bitcoin/testnet3/bitcoin.conf
```

## 👜 Creating a Descriptor Wallet

```bash
bitcoin-cli -named createwallet wallet_name=<ratedgwallet> descriptors=true disable_private_keys=true
bitcoin-cli loadwallet "<ratedgwallet>"
```

## 💾 Installing Electrum Wallet

```bash
wget <electrum-url>
wget <electrum-signature-url>

gpg --import ThomasV.asc
gpg --verify electrum-4.5.8-x86_64.AppImage.asc electrum-4.5.8-x86_64.AppImage

sudo apt install libfuse2 (if a FUSE error occurs)
chmod +x electrum-4.5.8-x86_64.AppImage
```

### Create Electrum Launcher

```bash
nano electrum.desktop
```

**Content of `electrum.desktop`:**

```ini
[Desktop Entry]
Version=1.0
Name=Electrum
Comment=Bitcoin Wallet
Exec=/home/<your-user>/Downloads/electrum-4.5.8-x86_64.AppImage --testnet
Icon=/home/<your-user>/electrum_logo.png
Terminal=false
Type=Application
Categories=Utility;Finance;
StartupNotify=true
```

```bash
sudo mv electrum.desktop /usr/share/applications/
sudo chmod +x /usr/share/applications/electrum.desktop
```

## 🔍 Importing Electrum Watch-Only Wallet into Bitcoin Core

### Step 1: Get Descriptor Info

```bash
bitcoin-cli getdescriptorinfo "wpkh([<fingerprint>/0h]<tpub>/0/*)"
```

You can convert `vpub` to `tpub` using:
👉 [https://jlopp.github.io/xpub-converter/](https://jlopp.github.io/xpub-converter/)

### Step 2: Import Descriptor

```bash
bitcoin-cli -rpcwallet=<ratedgwallet> importdescriptors '[
  {
    "desc": "wpkh([<fingerprint>/0h]<tpub>/0/*)#<checksum>",
    "range": [0, 1000],
    "timestamp": "now",
    "internal": false,
    "watchonly": true,
    "active": true
  }
]'
```

## 📬 Receiving Addresses & Verification

```bash
bitcoin-cli listdescriptors
bitcoin-cli help getnewaddress
bitcoin-cli getnewaddress "ratedgbech32" "bech32"
bitcoin-cli getaddressesbylabel "ratedgbech32"
bitcoin-cli getaddressinfo <address>
```

## 🧠 Creating Full Wallet (To Send to Watch-Only Wallet)

```bash
bitcoin-cli -named createwallet wallet_name="fullwallet" descriptors=true
bitcoin-cli -rpcwallet=fullwallet getwalletinfo
bitcoin-cli -rpcwallet=fullwallet listdescriptors
bitcoin-cli -rpcwallet=fullwallet listdescriptors true
```

### Create New Addresses

```bash
bitcoin-cli -rpcwallet=fullwallet getnewaddress "taprootaddr" "bech32m"
bitcoin-cli getaddressinfo <bech32m-address>

bitcoin-cli -rpcwallet=fullwallet getnewaddress "legacyaddr" "legacy"
bitcoin-cli getaddressinfo <legacy-address>
```

## 🧪 After Getting Testnet Coins

```bash
bitcoin-cli -rpcwallet=fullwallet getwalletinfo
bitcoin-cli -rpcwallet=fullwallet listunspent
bitcoin-cli -rpcwallet=fullwallet getbalance
bitcoin-cli -rpcwallet=fullwallet getunconfirmedbalance
```

## 💸 Sending Transactions

```bash
bitcoin-cli -rpcwallet=fullwallet sendtoaddress "<watchonly-address>" 0.00001
bitcoin-cli -rpcwallet=fullwallet listtransactions "*" 10
bitcoin-cli -rpcwallet=fullwallet gettransaction "<txid>" false true
```

## 🚫 Legacy Commands Not Applicable to Descriptor Wallets

> The following commands are **not compatible** with descriptor-based wallets:

1. `createwallet` with `descriptors=false`
2. `signmessage`
3. `dumpwallet`
4. `importwallet`
5. `dumpprivkey`
6. `verifymessage`

🧠 **Note**: Descriptor wallets don’t store keys in a way that allows traditional signing/verifying with addresses.

## 🧩 Reference: HD Derivation Concepts

- **BIP32** — Hierarchical Deterministic (HD) key derivation using `xpub/xprv`
- **BIP44 / BIP49 / BIP84 / BIP86** — Define standard derivation paths and address types
- Use consistent **indexes** to ensure deterministic address generation
