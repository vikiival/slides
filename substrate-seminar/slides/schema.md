# How does the schema look like?

<div grid="~ cols-2 gap-2" m="t-2">
<div>

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

</div>

<div>

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

</div>
</div>