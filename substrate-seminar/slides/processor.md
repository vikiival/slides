# Indexers 101

<div grid="~ cols-2 gap-2" m="t-2">
  <div>

  ## What does this thing do?

  - **Two types of data in each block** - Events and extrinsics
    - **Extrinsics** - The call that each user makes to the chain (`system.remark`).
    - **Events** - Interactions that are emmited by the chain (`balances.Transfer`).
  - **Rule of thumb** - It is more effective to process *events*.

  </div>

  <div>
    
  ## How does the processing works?


  1. For each block, extracts all requested extrinsics/events.
  2. For each event/extrinsic, call the pre-defined handler.
  3. Handler should transform the data to match the schema we defined.
  4. Save into the Postgres database.

  </div>
</div>