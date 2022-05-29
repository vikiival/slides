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

# OK, so what are resolvers?
and do we need them? 

- **Functions** - that expose certain data to the GraphQL schema.
- **Do not kill the user's device** - Parsing and processing data on the client-side is making the user's device slow.
- **Some queries are impossible to write** - Effectively, we can't write aggregation queries from GraphQL
- **Flexibility** - We can write queries that will return only the data we need.
- **No need for reindexing** - It's just a "server" extension.

---


# What's the current structure of Rubick?

- **mappings** - indexer's logic
- **model** - generated types  by subsquid
- **types** - our own types
- **server-extension** - folder for resolvers
  - *model* - how the data will look like in graphql
  - *query* - the logic to fetch the data from the DB (in SQL)
  - *resolvers* - function(s) that will be called when the query is executed

---

# Let's code <3

### Task 1 (issue #69)

write a query in SQL that returns number of NFTs that are
- parameters - `collectionId`
- should return `totalCount`

---

# Let's code <3

### Task 1 (How I would write it)

```sql
SELECT COUNT(*)
FROM nft_entity
WHERE collection_id = $1
AND price > 0;
```


---

# Let's code <3

### Task 2 (issue #29)

write a query in SQL that returns basic information about NFTs and events
- parameters - `interaction` type ('LIST', 'BUY', 'SEND')
- query should return `meta`, `timestamp`, `nft id`, `nft name`, `metadata image`, `event caller`, `nft issuer` per each event/NFT

---

# Let's code <3

### Task 2 (How I would write it)

```sql
SELECT e.meta, e.timestamp as date, ne.id, ne.name, me.image, ne.issuer, e.caller
FROM event e
LEFT JOIN nft_entity ne ON ne.id = e.nft_id
LEFT JOIN metadata_entity me ON me.id = ne.meta_id
WHERE e.interaction = 'LIST'
ORDER BY e.timestamp DESC
LIMIT 10
```

---

# A time for resolver (1)

We will make a resolver for the query from task 2.

We need to edit 3 files:

```bash
cd rubick/server-extension/
touch query/event.ts
touch model/event.model.ts
touch resolvers/nftEvents.ts
```
---

# A time for resolver (2)

touch query/event.ts

First, we need to add the query:

```ts
export const nftEventList = `SELECT e.meta, e.timestamp as date, ne.id, ne.name, me.image, ne.issuer, e.caller
FROM event e
LEFT JOIN nft_entity ne ON ne.id = e.nft_id
LEFT JOIN metadata_entity me ON me.id = ne.meta_id
WHERE e.interaction = $1
ORDER BY e.timestamp DESC
LIMIT $2`
```

---

# A time for resolver (2)

touch model/event.model.ts

Second, we have to create the model:

```ts
@ObjectType()
export class TokenEventEntity {
  @Field(() => String, { nullable: false })
  id!: string
  @Field(() => String, { nullable: true })
  name!: string
  @Field(() => Date, { nullable: false })
  date!: Date
  @Field(() => String, { nullable: true, defaultValue: '' })
  meta!: string
  @Field(() => String, { nullable: true, defaultValue: '' })
  image!: string
  @Field(() => String, { nullable: false })
  issuer!: string
  @Field(() => String,{ nullable: false })
  caller!: string
}
```

---

# A time for resolver (3)

touch resolvers/nftEvents.ts

Third, we finish the resolver:

```ts
@Resolver()
export class NFTEventResolver {
  constructor(private tx: () => Promise<EntityManager>) {}

  @Query(() => [TokenEventEntity])
  async nftEventListByType(
    @Arg('type', { nullable: false }) type: Interaction,
    @Arg('limit', { nullable: true, defaultValue: null }) limit: number,
  ): Promise<TokenEventEntity[]> {
    const result: TokenEventEntity[] = await makeQuery(this.tx, EventEntity, nftEventList, [type, limit])
    return result
  }
}
```

---

# A time for resolver (4)

touch resolvers/index.ts

Last we need to register our resolver:

```ts
export {
  NFTEventResolver
}
```

---

# Let's test it
Let's play on the playground

```bash
just build
just serve
```

and pass the following query:

```graphql
query MyQuery {
  nftEventListByType(type: "LIST", limit: 10) {
    id
    meta
    name
  }
}
```


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