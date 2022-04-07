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

# Into the KodaDot indexers

And how to make queries in GraphQL


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---


# Why do we need indexers?

How to make queries in GraphQL

- **Do not sacrifice UX** - Reading data from the blockchain takes a lot of time and resources.

- **Analytics** - making data views is a great way to get insights into the data.

- **You can't process RMRK** - RMRK is basically string in the blockchain, so without indexers you can't process it.

- **Simple GraphQL interface** - a great way to get data from the blockchain.


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What is indexer?

Tool to process Substrate blocks one by one.

- **Two types of data in each block** - Events and extrinsics
  - **Extrinsics** - The call that each user makes to the chain (`system.remark`).
  - **Events** - Interactions that are emmited by the chain (`balances.Transfer`).
- **Rule of thumb** - It is more effective to process *events*.

<!-- <div grid="~ cols-2 gap-2" m="t-2">



<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true">

</div> -->


<br>
<br>

---

# How does the processing works?


  1. For each block, extracts all requested extrinsics/events.
  2. For each event/extrinsic, call the pre-defined handler.
  3. Handler should transform the data to match the schema we defined.
  4. Save into the Postgres database.


<br>
<br>

---


# Which is indexers do we use?
And what are the differences?

<div grid="~ cols-2 gap-2" m="t-2">


<div>

- 📝 **SubQuery** - magick
  - Simplier one, less features
  - bit slower reindexing time
  - backbone of KodaDot's RMRK markteplace
  - so far *stable*
  - no ability for external calls
  - aggregations out of the box
  - auto-balanced for more nodes
</div>

<div>

- 🦑 **SubSquid** - rubick
  - More complex, higher flexibility
  - super fast reindexing (~ 2days)
  - can be used for external calls (fetch metadata from IPFS)
  - *alpha quality software*
  - can write custom queries through resolvers
  - europe based
  
</div>

</div>

<br>
<br>

---


# How does the schema look like? (1)


```graphql
type CollectionEntity @entity {
  version: String
  name: String
  max: Int!
  issuer: String
  symbol: String
  id: ID!
  metadata: String
  currentOwner: String
  nfts: [NFTEntity!] @derivedFrom(field: "collection")
  events: [CollectionEvent!]
  blockNumber: BigInt
  meta: MetadataEntity
  createdAt: DateTime!
}
```


---

# How does the schema look like? (2)


```graphql
type NFTEntity @entity {
  name: String
  instance: String
  transferable: Int
  collection: CollectionEntity!
  issuer: String
  sn: String
  id: ID!
  hash: String! @index
  metadata: String
  currentOwner: String
  price: BigInt!
  burned: Boolean!
  blockNumber: BigInt
  events: [Event!] @derivedFrom(field: "nft")
  emotes: [Emote!] @derivedFrom(field: "nft")
  meta: MetadataEntity
  createdAt: DateTime!
  updatedAt: DateTime!
}
```
---

# Let's query some data (1)

[GraphQL Bin for SubQuery](https://graphqlbin.com/)

[SubQuery Endpoint](https://kodadot.api.subquery.network)

### Task 1

Using **SubQuery** fetch the first my first collection with first 3 NFTs
- For collection I want: `id`, `name`
- For NFTs get: `id`, `name`, `currentOwner` and `price`
---

# Let's query some data (2)

[GraphQL Bin for SubQuery](https://graphqlbin.com/)

[SubQuery Endpoint](https://kodadot.api.subquery.network)

### Task 2 (try it yourself)

Using **SubQuery**  fetch the first my first `5` collections and just one NFTs
- For collection I want: `id`, `blockNumber`
- For NFTs get: `id`, `name`, `metadata` and `burned`
---

# Task 2 (how I would write it)

Using **SubQuery** fetch the first my first `5` collections and just one NFTs
- For collection I want: `id`, `currentOwner`, `blockNumber`
- For NFTs get: `id`, `name`, `metadata` and `burned`

```graphql
query myFiveCollections($account: String!) {
  collectionEntities(first: 5, filter: { currentOwner: { equalTo: $account } }) {
    nodes {
      id
      blockNumber
      nfts(first: 1) {
        nodes {
          id
          name
          metadata
          burned
        }
      }
    }
  }
}
```

---

# Let's query some data (3)

[GraphQL Bin for SubQuery](https://graphqlbin.com/)

[SubQuery Endpoint](https://kodadot.api.subquery.network)

### Task 3 (try it yourself)

Using **SubQuery**  fetch the first my first `5` nfts where given account is issuer but not the current owner
- For NFTs get: `id`, `name`, `currentOwner` and `issuer`
---

# Task 3 (how I would write it)

Using **SubQuery**  fetch the first my first `5` nfts where given account is issuer but not the current owner
- For NFTs get: `id`, `name`, `currentOwner` and `issuer`

```graphql
query fiveBoughtNFTs($account: String!) {
	nFTEntities(first:5 filter: { issuer: { equalTo: $account } currentOwner: { notEqualTo: $account } }) {
    nodes {
      id
      name
      currentOwner
      issuer
    }
  }
}
```

---

# Let's query some data (4)


[Built in SubSquid playground](https://app.gc.subsquid.io/beta/rubick/004/graphql)


### Task 4 (try it yourself)

Using **SubSquid**  fetch the first first `2` collections that has some not-burned and fetch `2` non-burned NFTs
- For collection I want: `id`, `name`
- For NFTs get: `name`
---

# Task 4 (how I would write it)

Using **SubSquid**  fetch the first first `2` collections that has some not-burned and fetch `2` non-burned NFTs
- For collection I want: `id`, `name`
- For NFTs get: `name`

```graphql
query collectionWithSomeNFTs  {
  collectionEntities(limit:2, where: {nfts_some: {burned_eq: false}}) {
    id
    name
    nfts(limit:2, where: { burned_eq: false }) {
      name
    }
  }
}

```

---

# Let's query some data (5)


[Built in SubSquid playground](https://app.gc.subsquid.io/beta/rubick/004/graphql)


### Task 5 (try it yourself)

Using **SubSquid**  fetch the first my first `5` nfts that has at least one buy
- For NFTs get: `id`, `name`, `metadata` and `burned`
- for meta get: `id`, `imag`
- for events get: `meta`, `caller`
---

# Task 5 (how I would write it)

Using **SubSquid**  fetch the first my first `5` nfts that has at least one buy
- For NFTs get: `id`, `name`, `metadata` and `burned`
- for meta get: `id`, `imag`
- for events get: `meta`, `caller`

```graphql
query atLeastOneTimeBoughtNFT {
  nFTEntities(limit:5, where: { events_some: { interaction_eq: BUY } }) {
    id
    meta {
      id
      image
    }
    events(where: { interaction_eq:BUY }) {
      meta
      caller
    }
  }
}
```



---
layout: center
class: text-center
---

# Links to know

[magick](https://github.com/vikiival/magick) · [rubick](https://github.com/kodadot/rubick) · [bounty queries](https://github.com/kodadot/nft-gallery/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Adataview)


---
layout: center
class: text-center
---

# Next session

Setting up an indexer