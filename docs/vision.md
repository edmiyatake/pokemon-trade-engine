# Goal

I want to build a platform where collectors can find the cards they’re looking for, put their own cards up for sale, and buy using cash, the value of cards they already own, or a mix of both.

I’m starting with a very simple version and growing it over time. The first goal is a platform where users can log in and browse every available card. Next, they’ll be able to add and view the cards they own. After that, they’ll see the monetary value of their collection. The final phase lets users initiate and complete trades using money and their cards.

## Views
- Sign Up / Log In Screen
- View All Cards
- Each card has an information page
- Search Bar or Page
- Collections View

## User Actions
- Create an account + delete, update
- Browse Pokemon cards (view current prices?) / Search for a specific card
- View information about a card
- Create a collection of cards that you own
- View Collections
- 

## Backlog of ideas that pop up in my brain as I write this
- Account Profiles / Reviews (Helps find scammers, you can view reviews before initiating a trade)
- A view that shows the most popular cards by view count
- Scan cards to add them
- Use stripe API if I wanted to make this a real project
- Profile that can be used to flex to friends (highest value card, collection, etc.)
- Landing page as if it were a sellable product
- Link cards to pokemon and pokemon each have their own page with information

## Stack
Off the top of my head, I know I need GraphQL because the goal of this project is to pull data from multiple sources efficiently. I want to use Next.js for the web application and MySQL as a 
database which holds important data. (Personal Choice, no architectural reason). 

Trades won’t be real. Verifying card value, ownership, and delivery would take a lot of extra work, so for now this is a simulation. Still, I think it’s an interesting idea, and it’s something I’d be willing to explore further in the future.

## Status

| Phase | Focus | Core concept | Status |
|---|---|---|---|
| 0 | Design | Outside-in API design | ⬜ |
| 1 | Catalog | Queries, resolvers, pagination | ⬜ |
| 2 | Accounts | First mutations, auth, context | ⬜ |
| 3 | Collections | Relationships, DataLoader | ⬜ |
| 4 | Collection value + price history | Field arguments, caching, query cost | ⬜ |
| 5 | Listings and trades | Authorization, transactions, state machines | ⬜ |
| 6 | Stretch | Subscriptions, AWS deployment | ⬜ |
 
---

## Phase 0: Design

**Goal:** A starting design for Phase 1

**Tasks**
- [ ] Create a list of functional requirements
- [ ] Sketch screens on paper and label the data each one shows (catalog with search bar, card info page)
- [ ] Write the query operation each screen would send
- [ ] Inspect tcgdex.dev responses for 3 or 4 cards
- [ ] Draw the ERD (entities, keys, indexes, JSON vs table decisions)
- [ ] Write `schema.graphql`
- [ ] Trace each query to the SQL it would trigger

---

## Phase 1: Catalog

**Goal:** Browse all cards, search by name, and view each card's info page with its current price.

**Backend (`api/`)**
- [ ] Scaffold with `npm init`, TypeScript, and GraphQL Yoga
- [ ] Run MySQL locally with Docker Compose
- [ ] Write SQL migration files for sets and cards
- [ ] Write a seed script that pulls from pokemontcg.io into MySQL, including each card's current price
- [ ] Keep the data source behind its own ingestion module so it can be swapped later
- [ ] Build a data access layer with `mysql2` and parameterized queries
- [ ] Resolvers: `cards` (search, set filter, cursor pagination), `card(id)`, `sets`
- [ ] Unit tests for cursor encoding and decoding
- [ ] Integration tests for each query against a test database
  **Frontend (`web/`)**
- [ ] `npx create-next-app@latest web`
- [ ] Set up Apollo Client or urql
- [ ] Catalog page with card grid, search bar, and "load more"
- [ ] Card info page
  **CI**
- [ ] GitHub Actions: lint, typecheck, and tests on every PR
 
---

## Phase 2: Accounts

**Goal:** Users can sign up, log in, log out, update their account, and delete it.

**Design first**
- [ ] Decide what deleting an account removes now and in later phases (collections, listings, open offers). Hard delete vs soft delete.
- [ ] Decide validation rules (unique email, password requirements) and how errors are returned
  **Backend**
- [ ] Add `users` and `sessions` tables
- [ ] Hash passwords with `argon2` or `bcrypt`
- [ ] Mutations: `signUp`, `logIn`, `logOut`, `updateAccount`, `deleteAccount`
- [ ] Session stored in an httpOnly cookie
- [ ] Resolve the current user once per request into GraphQL context
- [ ] Query `me` for the logged-in user
- [ ] Tests for every mutation, including failures (duplicate email, wrong password, logged out)
  **Frontend**
- [ ] Sign up and log in pages
- [ ] Account settings page (update, delete)
- [ ] Log out control and logged-in state in the header

---

## Phase 3: Collections

**Goal:** Logged-in users add cards they own and view their collection.

**Backend**
- [ ] Add a `collection_items` table
- [ ] Mutations: `addToCollection`, `updateQuantity`, `removeFromCollection`
- [ ] Query `myCollection`
- [ ] Apply the Phase 2 account deletion decision to collection items
- [ ] Log SQL queries, observe the N+1 problem, then fix it with DataLoader
- [ ] Record before and after query counts in `decisions.md`
  **Frontend**
- [ ] "Add to collection" control on card info pages
- [ ] Collection page with quantities

---

## Phase 4: Collection Value + Price History

**Goal:** Users see what their collection is worth and how card prices change over time.

**Backend**
- [ ] Add a `price_points` table (card, source, price, recorded_at)
- [ ] Scheduled ingestion script (cron locally, or a scheduled GitHub Action)
- [ ] Fields: `card.currentPrice`, `card.priceHistory(days: Int)`, `collection.totalValue`
- [ ] Batch price lookups with DataLoader
- [ ] Add query depth and complexity limits
- [ ] Add caching for catalog queries
  **Frontend**
- [ ] Total value on the collection page
- [ ] Price history chart on the card info page

---

## Phase 5: Listings and Trades

**Goal:** Users list cards for sale and negotiate trades using cards plus cash.

**Design first**
- [ ] Draw the offer state machine: `PENDING → COUNTERED → ACCEPTED / DECLINED / EXPIRED`
- [ ] List every invalid transition and how the API rejects it
- [ ] Decide what happens to a listing when one of its offers is accepted
  **Backend**
- [ ] Add a `listings` table with `createListing` and `cancelListing` mutations
- [ ] Query `listings` with filters (card, set, price range)
- [ ] Add `offers` and `offer_items` tables, with each offer tied to a listing
- [ ] Snapshot card valuations when each offer is created
- [ ] Mutations: `createOffer`, `counterOffer`, `acceptOffer`, `declineOffer`
- [ ] Authorization: only the two parties can act on an offer
- [ ] Wrap acceptance in a MySQL transaction that transfers card ownership
- [ ] Result union types for expected errors (`Offer | ValidationError`)
- [ ] Tests for every valid and invalid state transition
  **Frontend**
- [ ] Marketplace page to browse and search listings
- [ ] "List for sale" control on collection items
- [ ] Trade builder (pick cards on each side, add cash)
- [ ] Offer inbox with counter, accept, and decline actions

---

## Phase 6: Stretch

**Goal:** Real-time updates and a public deployment.

- [ ] Subscription `offerUpdated(offerId)` for live counteroffers
- [ ] Deploy MySQL on AWS RDS
- [ ] Deploy the API on AWS App Runner or ECS
- [ ] Deploy the web app
- [ ] Continuous deployment through GitHub Actions
- [ ] Price forecast: moving average plus trend, shown as a range

---

## Backlog

Ideas to pick up once their prerequisite phase is done.

| Idea | Earliest fit | What it teaches |
|---|---|---|
| Pokémon pages linked to cards | After Phase 3 | Consuming PokeAPI's GraphQL and joining two data sources (cards include Pokédex numbers) |
| Flex profile (highest value card, collection stats) | After Phase 4 | Aggregate queries over existing data |
| Most popular cards by view count | After Phase 4 | Designing writes triggered by reads without making queries side-effectful |
| Public profiles and reviews | After Phase 5 | Relationships and aggregates (average rating); reviews require completed trades |
| Landing page | Any time | Frontend polish and product presentation |
| Card scanning | Separate project | Image recognition |
| Stripe payments | Only if this becomes real | Payment flows and webhooks |
| Password reset | After Phase 2 | Token generation, expiry, and email delivery |
 