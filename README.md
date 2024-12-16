# Trust Wallet Issue: WalletConnect Integration

This project demonstrates a sample integration of WalletConnect. The issue being addressed relates specifically to the Trust Wallet in-app browser behavior when executing a `tron_signTransaction` request.

## Problem Statement

In the Trust Wallet mobile in-app browser, the tab closes immediately after a `tron_signTransaction` request is initiated, regardless of whether the transaction is approved or rejected. This abrupt closure prevents the application from completing the flow and displaying relevant status messages to the user. Notably, this behavior is exclusive to Trust Wallet’s in-app browser, as the flow works perfectly in desktop browsers.

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```
