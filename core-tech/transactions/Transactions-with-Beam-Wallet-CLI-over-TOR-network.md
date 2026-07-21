You can use the TOR network to hide your real IP during operation on the Beam CLI wallet.
This scenario is possible only for the CLI wallet connected to a remote node.
The TOR network will act as a proxy between a wallet and node.
This lets to anonymize the IP address of the BEAM Wallet machine for Node connected to.
At the moment this Tor support is implemented due to SOCKS5 proxy interface of Tor service.

Here are a few steps to set up an anonymous connection and hand on Beam:
1. Install TOR service on your machine (Browser or dedicated TOR service). The easiest way is to use browser installation.
2. Run a TOR browser or service. When you run a browser, tor service is started automatically.
3. Run BEAM Wallet CLI with "--proxy = true" and "--proxy_addr <tor service address>" arguments added. The wallet will work through the specified proxy server. Default TOR address (localhost: 9150) will be used in case when no "--proxy_addr" parameter specified.

Don't forget that Rescan command works only with its own node and will be impossible to use with TOR anonymization.