# base432v9import time
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

# Minimal ERC20 ABI methods
ERC20_METHODS = {
    "name()": "0x06fdde03",
    "symbol()": "0x95d89b41",
    "decimals()": "0x313ce567",
    "totalSupply()": "0x18160ddd",
}


def is_erc20(w3, address):
    try:
        for method in ERC20_METHODS.values():
            result = w3.eth.call({
                "to": address,
                "data": method
            })

            if result is None:
                return False

        return True

    except Exception:
        return False


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Detecting new ERC20 token deployments...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block > last_block:

                for block_num in range(last_block + 1, current_block + 1):

                    block = w3.eth.get_block(block_num, full_transactions=True)

                    for tx in block.transactions:

                        # contract deployment
                        if tx.to is None:

                            receipt = w3.eth.get_transaction_receipt(tx.hash)

                            contract = receipt.contractAddress

                            if contract and is_erc20(w3, contract):

                                print("🪙 New ERC20 Token")
                                print("Contract:", contract)
                                print("Deployer:", tx["from"])
                                print("Block:", block_num)
                                print("Tx:", tx.hash.hex())
                                print()

                last_block = current_block

            time.sleep(2)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
