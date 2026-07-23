- Search for key words `redirect, crypto, wallet, address`
- See 
```
showBitcoinQrCode() {
            this.dialog.open(el, {
                data: {
                    data: "bitcoin:1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm",
                    url: "./redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm",
                    address: "1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm",
                    title: "TITLE_BITCOIN_ADDRESS"
                }
            })
        }
```
- Enter the url http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
- Complete challenge