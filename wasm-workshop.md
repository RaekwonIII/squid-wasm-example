---
title: Introducing WASM Indexing
tags: Templates, Talk
description: View the slide with "Slide Mode".
slideOptions:
  center: false
  transition: fade
  spotlight:
    enabled: false
---
<!-- ![](https://i.imgur.com/MgLdaIs.jpg) -->

<!-- .slide: data-background="https://i.imgur.com/MgLdaIs.jpg" -->

<style>
    .left {
        text-align: left;
    }
    .smol {
        font-size: 24px;
    }
    .reveal pre {
        height: 100%;
        width: 100%;
    }
</style>

# WASM Indexing with Subsquid 🦑

https://hackmd.io/@RaekwonIII/wasm-indexing-subsquid

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

# *Enchanté* 👋

### Massimo Luraschi

### Developer Advocate @ Subsquid.io

* ![](https://i.imgur.com/wqhrc3N.png =24x24) [@RaekwonIII](https://twitter.com/RaekwonIII) 
* ![](https://i.imgur.com/kH2GXYx.png =24x24) RaekwonIII#3962 
* ![](https://i.imgur.com/92AwE1H.png =24x24) [@RaekwonTheChefIII](https://t.me/RaekwonTheChefIII) 
* ![](https://i.imgur.com/MGAfGvC.png =24x24) [RaekwonIII](https://github.com/RaekwonIII) 

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

## Subsquid: Web3's Best Data Backend Framework

* Framework for robust, natively multichain APIs and analytics
  * Hosted service
* Available on 33 networks (and counting)
* Focused on customisation and performance
* Built to scale

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

![](https://i.imgur.com/nmBzrEb.png)

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

## Resources

* [Subsquid Docs](https://docs.subsquid.io)
* [Moonbeam docs](https://docs.moonbeam.network/builders/integrations/indexers/subsquid/)
* [Astar docs](https://docs.astar.network/integration/indexers/subsquid)
* [Template project](https://github.com/subsquid/squid-template) ([projects](https://github.com/subsquid/squid-evm-template))
* [Aquarium](https://app.subsquid.io/aquarium)
* [Stackexchange](https://substrate.stackexchange.com/) (subsquid tag):
* Community: [Telegram](https://t.me/HydraDevs) group, [Discord](https://discord.gg/dxR4wNgdjV) server 

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="left" -->

## New release: Fire Squid 🔥🦑

  * Batch call unwrapping (nasty `batch_all` 😡)
  * Lazy transactions
  * Batch processing
  * Context field selector
  * Secrets in environment variables
  * Two gateways (exploration vs processor)
  * **WASM contract support**

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

## Let's get hacking 🛠️🦑

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

## Goal: ERC20 tracker

1. Setup<!-- .element: class="fragment" -->
2. Review the schema<!-- .element: class="fragment" -->
3. Generate models<!-- .element: class="fragment" -->
4. Generate Interfaces<!-- .element: class="fragment" -->
5. Implement logic in Processor<!-- .element: class="fragment" -->
6. Launch database container<!-- .element: class="fragment" -->
7. Create and apply database migration<!-- .element: class="fragment" -->
8. Launch processor and GraphQL server<!-- .element: class="fragment" -->

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Setup

* Requisites: [Node.js](https://nodejs.org/en/download/) (16 or later), [Docker](https://docs.docker.com/get-docker/)

* GitHub [template](https://github.com/subsquid/squid-template), *Use this template*, then

```bash
git clone git@github.com:<account>/squid-template.git
```

* Install dependencies from project's root folder

```bash
cd squid-template && npm i && npm update
``` 

* Install additional dependencies

```bash
npm i @subsquid/ink-abi @subsquid/ink-typegen
```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Codegen (schema)

Define entities we want to track in `schema.graphql` file in the project root folder

* Token (and token transfers)
* Ownership of tokens
* Contracts and their minted tokens

```graphql=
type Owner @entity {
  id: ID!
  balance: BigInt!
}

type Transfer @entity {
  id: ID!
  from: Owner
  to: Owner
  amount: BigInt!
  timestamp: DateTime!
  block: Int!
}

```

----

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Codegen (proj structure)

* From project's root folder, launch 

```bash
make codegen
```

* New files will be created ![](https://i.imgur.com/xv6AJE9.png =x450)

----

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Codegen (classes)

### Let's take the `Owner` entity as an example

```typescript=
import {Entity as Entity_, Column as Column_, PrimaryColumn as PrimaryColumn_} from "typeorm"
import * as marshal from "./marshal"

@Entity_()
export class Owner {
  constructor(props?: Partial<Owner>) {
    Object.assign(this, props)
  }

  @PrimaryColumn_()
  id!: string

  @Column_("numeric", {transformer: marshal.bigintTransformer, nullable: false})
  balance!: bigint
}

```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# WASM Typegen

* We deployed an ERC20 INK contract on Shybuia [from Parity examples](https://github.com/paritytech/ink/tree/604976c239dcad62d2f23f754cd45d4ab2c58d8c/examples/erc20)
* We need the ERC20 ABI to be able to process its logs
* Copy it from [here](https://raw.githubusercontent.com/subsquid/squid/42d07b0d01b02ada4b28f057ada3b05aa762a170/test/shibuya-erc20/metadata.json) (or source it elsewhere)
* Paste it into a file named `ERC20.json` in `src`
* From project root folder, launch
```bash
npx squid-ink-typegen --abi src/erc20.json --output src/erc20.ts
```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Processor logic

Processor logic needs to be updated, need to introduce WASM contract decoding and write data to new entities:

```typescript=
import * as ss58 from "@subsquid/ss58"
import {BatchContext, BatchProcessorItem, SubstrateBatchProcessor} from "@subsquid/substrate-processor"
import {Store, TypeormDatabase} from "@subsquid/typeorm-store"
import {In} from "typeorm"
import * as erc20 from "./erc20"
import {Owner, Transfer} from "./model"


const CONTRACT_ADDRESS = '0x5207202c27b646ceeb294ce516d4334edafbd771f869215cb070ba51dd7e2c72'


const processor = new SubstrateBatchProcessor()
    .setDataSource({
        archive: "https://shibuya.archive.subsquid.io/graphql"
    })
    .addContractsContractEmitted(CONTRACT_ADDRESS, {
        data: {
            event: {args: true}
        }
    } as const)


type Item = BatchProcessorItem<typeof processor>
type Ctx = BatchContext<Store, Item>


processor.run(new TypeormDatabase(), async ctx => {
    let txs = extractTransferRecords(ctx)

    let ownerIds = new Set<string>()
    txs.forEach(tx => {
        if (tx.from) {
            ownerIds.add(tx.from)
        }
        if (tx.to) {
            ownerIds.add(tx.to)
        }
    })

    let owners = await ctx.store.findBy(Owner, {
        id: In([...ownerIds])
    }).then(owners => {
        return new Map(owners.map(o => [o.id, o]))
    })

    let transfers = txs.map(tx => {
        let transfer = new Transfer({
            id: tx.id,
            amount: tx.amount,
            block: tx.block,
            timestamp: tx.timestamp
        })

        if (tx.from) {
            transfer.from = owners.get(tx.from)
            if (transfer.from == null) {
                transfer.from = new Owner({id: tx.from, balance: 0n})
                owners.set(tx.from, transfer.from)
            }
            transfer.from.balance -= tx.amount
        }

        if (tx.to) {
            transfer.to = owners.get(tx.to)
            if (transfer.to == null) {
                transfer.to = new Owner({id: tx.to, balance: 0n})
                owners.set(tx.to, transfer.to)
            }
            transfer.to.balance += tx.amount
        }

        return transfer
    })

    await ctx.store.save([...owners.values()])
    await ctx.store.insert(transfers)
})


interface TransferRecord {
    id: string
    from?: string
    to?: string
    amount: bigint
    block: number
    timestamp: Date
}


function extractTransferRecords(ctx: Ctx): TransferRecord[] {
    let records: TransferRecord[] = []
    for (let block of ctx.blocks) {
        for (let item of block.items) {
            if (item.name == 'Contracts.ContractEmitted' && item.event.args.contract == CONTRACT_ADDRESS) {
                let event = erc20.decodeEvent(item.event.args.data)
                if (event.__kind == 'Transfer') {
                    records.push({
                        id: item.event.id,
                        from: event.from && ss58.codec(5).encode(event.from),
                        to: event.to && ss58.codec(5).encode(event.to),
                        amount: event.value,
                        block: block.header.height,
                        timestamp: new Date(block.header.timestamp)
                    })
                }
            }
        }
    }
    return records
}

```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Database

* Squid APIs need a database to store processed data
* Templates have `docker-compose.yml` file to launch a container
* From project's root folder, launch 
```bash
docker-compose up -d
```
* or, alternatively
```bash
make up
```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Create and apply database migration

* Build code

```bash=
npm run build
```

* Remove existing migrations

```bash=
rm -rf db/migrations/*js
```

* Create and apply new migration

```bash=
npx squid-typeorm-migration generate
make migrate
```

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

<!-- .slide: class="smol" -->
<!-- .slide: class="left" -->

# Launch the project

The project consists of a Processor (which we just implemented) and a GraphQL server.

* Launch the Processor (this will lock the console window)

```bash=
node -r dotenv/config lib/processor.js # make process
```

* Launch the GraphQL server (in another console window)

```bash=
npx squid-graphql-server # make serve
```

* Open the browser at http://localhost:4350/graphql

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

## Good news everyone!

![](https://media.giphy.com/media/3zFcbgHoIXzykQc7vU/giphy.gif)

There ~~is~~ will be a [WASM template](https://github.com/RaekwonIII/squid-wasm-example)

And this project ~~is~~ will be available in our [Aquarium](https://app.subsquid.io/aquarium/Moon-Substrate-EVM)

---

<!-- .slide: data-background="https://github.com/RaekwonIII/moonbeam-workshop/raw/main/base_bg.png" -->

# Thank you 🦑


Follow the project on GitHub
https://github.com/subsquid/squid

![](https://media.giphy.com/media/qUIm5wu6LAAog/giphy.gif)

Give us a ⭐, would you?
