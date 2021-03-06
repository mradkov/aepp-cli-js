





  

```js
#!/usr/bin/env node

```







# æternity CLI `AENS` file

This script initialize all `AENS` function


  

```js
/*
 * ISC License (ISC)
 * Copyright (c) 2018 aeternity developers
 *
 *  Permission to use, copy, modify, and/or distribute this software for any
 *  purpose with or without fee is hereby granted, provided that the above
 *  copyright notice and this permission notice appear in all copies.
 *
 *  THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH
 *  REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY
 *  AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
 *  INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM
 *  LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR
 *  OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
 *  PERFORMANCE OF THIS SOFTWARE.
 */

import { initClientByWalletFile } from '../utils/cli'
import { printError, print, printUnderscored } from '../utils/print'
import { handleApiError } from '../utils/errors'
import { isAvailable, updateNameStatus, validateName } from '../utils/helpers'


```







## Claim `name` function


  

```js
async function claim (walletPath, domain, options) {

```







Parse options(`ttl`, `nameTtl`)


  

```js
  const ttl = parseInt(options.ttl)
  const nameTtl = parseInt(options.nameTtl)
  try {

```







Validate `name`(check if `name` end on `.test`)


  

```js
    validateName(domain)


```







Get `keyPair` by `walletPath`, decrypt using password and initialize `Ae` client with this `keyPair`


  

```js
    const client = await initClientByWalletFile(walletPath, options)

    await handleApiError(async () => {

```







Check if that `name' available


  

```js
      const name = await updateNameStatus(domain)(client)
      if (!isAvailable(name)) {
        print('Domain not available')
        process.exit(1)
      }


```







Create `preclaimName` transaction


  

```js
      const { salt, height } = await client.aensPreclaim(domain, { nameTtl, ttl })
      print('Pre-Claimed')


```







Wait for next block and create `claimName` transaction


  

```js
      await client.aensClaim(domain, salt, (height + 1), { nameTtl, ttl })
      print('Claimed')


```







Update `name` pointer


  

```js
      const { id } = await updateNameStatus(domain)(client)
      const { hash } = await client.aensUpdate(id, await client.address(), { nameTtl, ttl })
      print('Updated')

      print(`Name ${domain} claimed`)
      printUnderscored('Transaction hash', hash)
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







##Transfer `name` function


  

```js
async function transferName (walletPath, domain, address, options) {

```







Parse options(`ttl`, `nameTtl` and `nonce`)


  

```js
  const ttl = parseInt(options.ttl)
  const nameTtl = parseInt(options.nameTtl)
  const nonce = parseInt(options.nonce)

  if (!address) {
    program.outputHelp()
    process.exit(1)
  }
  try {

```







Get `keyPair` by `walletPath`, decrypt using password and initialize `Ae` client with this `keyPair`


  

```js
    const client = await initClientByWalletFile(walletPath, options)

    await handleApiError(async () => {

```







Check if that `name` is unavailable and we can transfer it


  

```js
      const name = await updateNameStatus(domain)(client)
      if (isAvailable(name)) {
        print(`Domain is available, nothing to transfer`)
        process.exit(1)
      }


```







Create `transferName` transaction


  

```js
      const transferTX = await client.aensTransfer(name.id, address, { ttl, nameTtl, nonce })
      print('Transfer Success')
      printUnderscored('Transaction hash', transferTX.hash)
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







##Update `name` function


  

```js
async function updateName (walletPath, domain, address, options) {

```







Parse options(`ttl`, `nameTtl` and `nonce``)


  

```js
  const ttl = parseInt(options.ttl)
  const nameTtl = parseInt(options.nameTtl)
  const nonce = parseInt(options.nonce)

  if (!address) {
    program.outputHelp()
    process.exit(1)
  }

  try {

```







Get `keyPair` by `walletPath`, decrypt using password and initialize `Ae` client with this `keyPair`


  

```js
    const client = await initClientByWalletFile(walletPath, options)

    await handleApiError(async () => {

```







Check if that `name` is unavailable and we can update it


  

```js
      const name = await updateNameStatus(domain)(client)
      if (isAvailable(name)) {
        print(`Domain is ${name.status} and cannot be transferred`)
        process.exit(1)
      }


```







Create `updateName` transaction


  

```js
      const updateNameTx = await client.aensUpdate(name.id, address, { ttl, nameTtl, nonce })
      print('Update Success')
      printUnderscored('Transaction Hash', updateNameTx.hash)
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}


```







##Revoke `name` function


  

```js
async function revokeName (walletPath, domain, options) {

```







Parse options(`ttl` and `nonce`)


  

```js
  const ttl = parseInt(options.ttl)
  const nonce = parseInt(options.nonce)

  try {

```







Get `keyPair` by `walletPath`, decrypt using password and initialize `Ae` client with this `keyPair`


  

```js
    const client = await initClientByWalletFile(walletPath, options)

    await handleApiError(async () => {

```







Check if `name` is unavailable and we can revoke it


  

```js
      const name = await updateNameStatus(domain)(client)
      if (isAvailable(name)) {
        print(`Domain is available, nothing to revoke`)
        process.exit(1)
      }


```







Create `revokeName` transaction


  

```js
      const revokeTx = await client.aensRevoke(name.id, { ttl, nonce })
      print('Revoke Success')
      printUnderscored('Transaction hash', revokeTx.hash)
    })
  } catch (e) {
    printError(e.message)
    process.exit(1)
  }
}

export const AENS = {
  revokeName,
  updateName,
  claim,
  transferName
}


```




