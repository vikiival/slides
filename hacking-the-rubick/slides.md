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

# Hacking the rubick

How to make GraphQL resolvers in SubSquid indexer


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---


# Recap: Why do we need indexers?

How to make queries in GraphQL

- **Do not sacrifice UX** - Reading data from the blockchain takes a lot of time and resources.

- **Analytics** - making data views is a great way to get insights into the data.

- **You can't process RMRK** - RMRK is basically string in the blockchain, so without indexers you can't process it.

- **Simple GraphQL interface** - a great way to get data from the blockchain.


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Recap: What is indexer?

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

# Recap: How does the processing works?


  1. For each block, extracts all requested extrinsics/events.
  2. For each event/extrinsic, call the pre-defined handler.
  3. Handler should transform the data to match the schema we defined.
  4. Save into the Postgres database.


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

# How does the schema look like? (3)

```graphql
type Event implements EventType @entity {
  id: ID!
  blockNumber: BigInt
  timestamp: DateTime!
  caller: String!
  currentOwner: String! # currentOwner
  interaction: Interaction!
  meta: String!
  nft: NFTEntity!
}
```
---

# Let's query some data


[Built in SubSquid playground](https://app.gc.subsquid.io/beta/rubick/006/graphql)


### Example Task

Using **rubick** fetch first `10` NFTs that has been recently listed
- For event I want: `meta`, `timestamp`
- For NFT get: `id`, `name`
---

# Example Task (how I would write it)

Using **rubick** fetch first `10` NFTs that has been recently listed
- For event I want: `meta`, `timestamp`
- For NFT get: `id`, `name`

```graphql
query lastNftListByEvent($limit: Int!, $event: Interaction!) {
  events(
    limit: $limit
    where: { interaction_eq: $event, nft: {burned_eq: false} }
    orderBy: timestamp_DESC
  ) {
    meta
    timestamp
    nft {
      id
      name
    }
  }
}

```

---

# OK, so why do we need resolvers? 

- **Do not kill user's device** - Parsing and processing data on the client side is making the user's device slow.
- **Some quries are impossible to write** - Effectively, we can't write aggregation queries from GraphQL
- **Flexibility** - We can write queries that will return only the data we need.

---


# What's the current structure of rubick?

- **mappings** - indexer's logic
- **model** - generated types  by subsquid
- **types** - our own types
- **server-extension** - folder for resolvers
  - *model*
  - *query*
  - *resolvers*

---

# Let's code <3

### Task 1 (issue #69)

resolver to check how many NFTs are listed
- parameters - `collectionId`
- should return `totalCount`

---

# Let's code <3

### Task 1 (How would I write it)

resolver to check how many NFTs are listed
- parameters - `collectionId`
- should return `totalCount`

---

layout: center
class: text-center
---

# Links to know

[rubick](https://github.com/kodadot/rubick) · [bounty queries](https://github.com/kodadot/nft-gallery/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3Adataview)


---
layout: center
class: text-center
---

# Next session

Index your own data