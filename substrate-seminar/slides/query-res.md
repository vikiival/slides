# Let's query some data

### Task (how I would write it)

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