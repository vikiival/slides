# Why is Uniquery cool?

<div grid="~ cols-2 gap-2" m="t-2">
<div>
<h2>before</h2>

```graphql
  query nftListByCollectionId {
    nft: nftEntities(where: {
      collection: {id_eq: "2305670031"}}
  ) {
      id
      metadata
      currentOwner
      issuer
    }
  }
```

</div>
  <div>
  <h2>after</h2>

  ```ts
  import { getClient, ask } from '@kodadot1/uniquery'

  // using builder
  const client = getClient()
  const id = '2305670031'
  const query = client.itemListByCollectionId(id)

  // using REST
  const path = `/nftListByCollectionId/${id}`
  const result = await ask(path)

  ```

  </div>
</div>