## FOR ONCHAIN SUMMER BUILDATHON

MULTISIGNATURE WALLET CONTARCT ADDRESS: `0x1EaDbE06B4cb810db8274EfCdF3A47d5AF16Eec1`
MULTISIGNATURE CONTRACT ADDRESS: `0x426c1C9a5929526DC8b77b1EDC02aD8AB5fC22fb`


---

Example how to integrate Coinbase Smart Wallet (RainbowKit, Wagmi)

```tsx
  const projectId = "00000000";
  const chainsSet = [base]
  coinbaseWallet.preference = "smartWalletOnly"

  const config = getDefaultConfig({
    appName: 'DONAT3.LIVE',
    appIcon: "https://donat3.live/images/donat3-logo.png",
    projectId: projectId,
    chains: chainsSet,
    multiInjectedProviderDiscovery: false,
    transports: {
      [mainnet.id]: http(),
      [dfkTestnet.id]: http(),
      [dfkMainnet.id]: http(),
      [bsc.id]: http(),
      [bscTestnet.id]: http(),
      [klaytnMainnet.id]: http(),
      [klaytnTestnet.id]: http(),
      [sepolia.id]: http(),
      [wemixMainnet.id]: http(),
      [wemixTestnet.id]: http(),
      [polygon.id]: http(),
      [polygonMumbai.id]: http(),
      [jibchain.id]: http(),
      [hardhat.id]: http(),
      [avalanche.id]: http(),
      [base.id]: http(),
    },
    wallets: [{
      groupName: "Recommended", wallets: [injectedWallet, metaMaskWallet, coinbaseWallet, walletConnectWallet]
    }],

  })
```

---

Example of code to activate Coinbase Smart Wallet

```tsx
  import { useCapabilities, useWriteContracts } from "wagmi/experimental"

  const { data: availableCapabilities } = useCapabilities({ account: account.address });

  const hasWalletCapabilities = (availableCaps, chainId) => {
    if (!availableCaps || !chainId) return false;
    const capabilitiesForChain = availableCaps[chainId];
    if (
      capabilitiesForChain["paymasterService"] &&
      capabilitiesForChain["paymasterService"].supported
    ) {
      return true;
    } else {
      return false;
    }
  }

  const memoizedHasWalletCapabilities = useMemo(() => {
    return hasWalletCapabilities(availableCapabilities, account.chainId);
  }, [availableCapabilities, account.chainId]);

  const capabilities = useMemo(() => {
    if (memoizedHasWalletCapabilities) {
      return {
        paymasterService: {
          url: paymasterUrl
        },
      };
    }
  }, [availableCapabilities, account.chainId]);

  const { writeContracts: writeWalletContracts } = useWriteContracts({
    mutation: {
      onError: (error) => {
        onError && onError();
      },
      onSuccess: (data) => {
        setCompleteDonate(true);
        setWalletData(data);
      }
    }
  });
  
  const handleDonate = async (event: React.FormEvent) => {
    event.preventDefault();
    if (memoizedHasWalletCapabilities) {
      writeWalletContracts({
        contracts: [
          {
            abi: erc20Abi,
            args: [coin?.data.walletContractAddress, parseUnits(amount, coin?.data.decimalPlaces)],
            functionName: 'approve',
            address: coin?.data.address,
          },
          {
            abi: walletAbi,
            functionName: "deposit",
            address: coin?.data.walletContractAddress,
            args: [coin?.data.address, parseUnits(amount, coin?.data.decimalPlaces), refCode]
          }
        ],
        capabilities,
      });
    }
  });
```