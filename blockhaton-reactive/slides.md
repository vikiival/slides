---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Into into Reactive NFTs

How to make implement them in your marketplace?


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---


# What is KodaDot in the first place?

Multichain, open-source NFT marketplace built as public good for Dotsama Ecosystem.

- **Multichain** - Supporting Kusama, Statemine and Balilisk (more to come).

- **Open-source** - 190 forks 🍴, 265 stars ⭐️, 70+ contributors 👯.

- **Standard agnostic** - From custom pallets to contracts, we support them under the one codebase.


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What's the tech behind?

The techstack we use.

- **Frontend** - NuxtJS (v2.x), statically generated, deployable from Netlify to IPFS

- **"Backend"** - Indexers from SubQuery and SubSquid that expose a GraphQL API. 

- **Microserivces** - Cloudflare workers for a critical part of the system (pinning, caching, etc).

- **Libraries** - Lot of NPM modules (under @kodadot1)



<!-- <div grid="~ cols-2 gap-2" m="t-2">



<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true">

</div> -->


<br>
<br>

---

# Let's build something reactive marketplace

From the Reactive stack we will use today:

1. Conditional rendering (library)

2. Generative art (high PoC)


<br>
<br>

---


# Conditional rendering (1)

We use a JSON logic to update the state of the NFT

```json
[
  { "===": ["Fksmad33PFxhrQXNYPPJozgWrv82zuFLvXK7Rh8m1xQhe98", "Fksmad33PFxhrQXNYPPJozgWrv82zuFLvXK7Rh8m1xQhe98"] },
  "metadata",
  "ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru"
]
```


---

# Conditional rendering (2)


We use can also use variables

```json
[
    { "===": [{ "var": "currentOwner" }, { "var": "issuer" }] },
    "metadata",
    "ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru"
]
```
---

# Conditional rendering (3)

Also works for nested objects

```json
[
    { "===": [{ "var": "currentOwner" }, { "var": "issuer" }] },
    "meta.image",
    "ipfs://ipfs/bafybeiennmmolv3keadouvgrhqkkfhprobvikl6hmblg4x2nxsn7s7lrjq"
]
```
---

# Conditional rendering (4)

The object we will use for today

```ts
export type SomeNFT = {
  blockNumber: string
  burned: boolean
  currentOwner: string
  id: string
  issuer: string
  metadata: string
  name: string
  price: string
  recipient: string
  royalty: number
  sn: string
  meta: {
    description: string
    name: string
    image: string
  }
}
```
---

# Let's do something

First we need to install the library

```bash
yarn add @kodadot1/if-that
```


### Example Task (1)

Using **if-that** I want to set new name of NFT if currentOwner is equal to Viki's address.

- Name should be changed to: `Hello Viki @ Blockhaton`
---

# Example Task (how I would write it)

Using **if-that** I want to set new name of NFT if currentOwner is equal to Viki's address.
- Name should be changed to: `Hello Viki @ Blockhaton`

first we need to import the library and some sample data

```ts
import { IfThat, eq, build, gt, lte } from '@kodadot1/if-that'
import { nftList, SomeNFT } from './data'
```

Now apply the rules

```ts
const newName = 'Hello Viki @ Blockhaton'
const address = 'Fksmad33PFxhrQXNYPPJozgWrv82zuFLvXK7Rh8m1xQhe98'
const ifThat: IfThat<SomeNFT> = [eq(nft.currentOwner, address), 'name', newName]
const res = build([ifThat], nft)
expect(res.name).eq(newName)
expect(nft.name).not.eq(res.name)
```

---

# Let's do something

### Example Task (2)

Using **if-that** I want to change the metadata if the current owner does not equal to Viki's address and royalty is set to more than 26.

- Metadata should be changed to: `ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru`
---

# Example Task (how I would write it)

Using **if-that** I want to change the metadata if the current owner does not equal to Viki's address and royalty is set to more than 26.
- Metadata should be changed to: `ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru`

```ts
import { IfThat, neq, build, gt, lte } from '@kodadot1/if-that'
import { nftList, SomeNFT } from './data'
```

Now apply the rules

```ts
const newMeta = 'ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru'
const address = 'Fksmad33PFxhrQXNYPPJozgWrv82zuFLvXK7Rh8m1xQhe98'
const ifThat: IfThat<SomeNFT> = [[neq(nft.currentOwner, address), lte(nft.royalty, 26)], 'metadata', newMeta]
const res = build([ifThat], nft)
expect(res.metadata).eq(newMeta)
```

---

# OK, so how to integrate into my NFT?
Let's make a rule

```ts
import { IfThat, eq, asVar, Variable } from '@kodadot1/if-that'
import { SomeNFT } from './data'
const ifThat: IfThat<SomeNFT> = [eq(asVar('currentOwner'), asVar('issuer')), 'metadata', newMeta]
```

- add this logic to the JSON metadata
- upload to IPFS
- when you read the metadata, check if it contains the logic field
- if it does, then you can use the logic to update the state of the NFT

---


# The JSON result

```json
{
  "name": "Sunspot",
  "description": "Astrophysicists estimate the likelihood of a solar storm is 1",
  "image": "ipfs://ipfs/bafkreigutyg3pg2bjjfah5fccp2x65lefte763ps4dwfwaig3ej7kwogh4",
  "external_url": "https://kodadot.xyz",
  "type": "image/png",
  "logic": [
    { "===": [{ "var": "currentOwner" }, { "var": "issuer" }] },
    "metadata",
    "ipfs://ipfs/bafkreie6aheicniw2ene6iofxj7e6cettjioxojdstwabdqkxozjk2tyru"
  ]
}
```

---

# Where to test it

KodaDot has a sandbox for testing




---
layout: center
class: text-center
---

# Links to know

[nft-gallery](https://github.com/kodadot/nft-gallery) · [packages](https://github.com/kodadot/packages) · [vikiival](https://twitter.com/vikiival)


---
layout: center
class: text-center
---

# Next session

Index your own data