# Activating TEL Issuance at the Application Layer

This codebase is designed to calculate and distribute TAN incentives, establishing the platform's application layer incentive mechanisms. These mechanisms must be voted on and approved by the TAN council.

With 194.44M TEL in the TAN Council Safe, weekly distributions of TEL will flow to stakers based on their share of platform production. This initiates the unified staking model and activates essential platform adoption incentives.

## Motivation

The Telcoin Platform requires active application layer participation to achieve its vision of universal mobile-enabled blockchain services. By implementing TEL issuance for stakers, we aim to:

- **Drive user adoption through staker referrals**
- **Foster an on-chain ecosystem aligned with natural market incentives**
- **Empower the entire value chain with ownership rights**, fulfilling the Association's constitutional mandate for inclusive, decentralized governance

While multiple incentive programs are planned for the future, the Staker Incentives Calculator implementation, which issues incentives based on the TANIP-1 referral program, is currently in production. A future issuance program in development is the Developer incentives, which will aim to encourage GSMA member application development.

## Staker Issuance Specification

### TEL Distribution Framework

- **TAN Council Safe Balance:** 194.44M TEL
- **Weekly Distribution:**
  - **Stakers:** 3,205,128.205 TEL
- **Distribution Timing:** Every Wednesday at 00:00 UTC

### Staker Rules

- **Eligibility:** All Telcoin Developer apps' users' who have added another user’s referral code and staked
- **Allocation Formula:** 3,205,128.205 TEL per week
- **Basis for Income Formula:**
  - Pro-rata share of:
    - Personal fees paid
    - Referred users' fees paid
- **TAN Maximum Weekly TEL Issuance Rule:** Throughout the entire period, total staked TEL must exceed (Lifetime TEL issuance + Current week TEL issuance)

## Viewing the official rewards

For community members wishing to view the official published rewards for each TANIP-1 period (weeks starting Feb 19 at 00:00 UTC), refer to the `rewards` directory and consult the file for the period, for example `rewards/staker_rewards_period_0.xlsx` or `rewards/staker_rewards_period_0.json`. These rewards files list the supported networks, start and end blocks, and a map of each address to its rewards for the period.

### Note that the rewards amounts are denominated using human-readable format in the Excel file but using EVM-recognizable numbers in the JSON file so the TEL decimals transformation must be applied to the JSON. This means all listed amounts in the JSON must be divided by 100 to get the reward amount in TEL which has a decimal precision of 2.

## Running the Tests

This codebase uses the Jest framework for TypeScript testing. Tests exist for each of the calculators; however, as mentioned, the StakerIncentivesCalculator is currently the only calculator in production, pending the approval of more TANIPs.

To run the tests, first ensure you are using the correct node version:

```bash
nvm use 18
```

Install dependencies:

```bash
yarn install
```

Make sure to edit `src/config.ts` to fit your requirements, such as the staker incentives amounts, and set the `.env` with RPC endpoints.

The `src/test/StakerIncentivesCalculatorFork.test.ts` features an extensive end-to-end fork simulation test of an entire period's worth of staking, TAN trading, and user fees; these actions are fuzzed using randomly generated users, stakes, fees, and referral relationships. Because this test executes on-chain actions on a local fork, it requires running a local anvil fork of the Polygon mainnet alongside the test.

To run an Anvil fork of Polygon, make sure you have Foundry installed and then execute:

```bash
anvil --fork-url $POLYGON_RPC_URL
```

If Foundry is not installed or the fork test is not desired (~20s), disable it by adding `.skip` to line 171, i.e.:

```javascript
it.skip("should return the correct reward amounts per staker and referrer", async () => {
```

Then run all the tests:

```bash
yarn jest
```

## Running the StakerIncentivesCalculator

The program's entry point is the `src/app.ts` file, which instantiates all the data sources and fetches required data to run the `StakerIncentivesCalculator`. Future additions to the program, such as new incentive program calculators, will be added to this file. For a deeper dive into the staker issuance incentives calculator, including specification, invariants, and system diagrams, refer to the `staker-incentives-calculator.md` file

Ensure all required local files for the data sources relevant to the calculators being run are present in the expected locations. The TAN Staker Incentives Program calculator does not require any of these because it fetches all required data from on-chain, but future calculators may rely on the local files, such as those using the `UserRegistry`, which expects a `users_wallets_referrals.json` file generated by the TAN backend.

First, ensure you are using the correct node version:

```bash
nvm use 18
```

Install dependencies:

```bash
yarn install
```

Then build the project:

```bash
yarn build
```

Now you can run the main application. The program accepts the chain(s) to fetch and process information from as well as an optional start block and endblock, for example:

```bash
# calculate based on Polygon data only, starting from block 67070000 until block 67078000
yarn start polygon=67070000:67078000

# calculate based on Polygon data, starting from the TANIssuanceHistory's stored `lastSettlementBlock`
yarn start polygon

# calculate based on Polygon and Mainnet
yarn start polygon mainnet

# calculate based on Polygon and Mainnet with specified start blocks for each.
yarn start polygon=67070000:67078000 mainnet=21740000:21739790
```

### Onchain Deployments used by the StakerIncentivesCalculator

The full list of contracts used by the calculator can be found in `staker-incentives-calculator.md`, though the main two relied on are as follows:

#### **Staking Plugins**

- `0xCAa823Fd48bec0134c8285Fd3C34F9D95CF3280f`

#### **TANIssuanceHistory**

- `0xE533911F00f1C3B58BB8D821131C9B6E2452Fc27`

### Note on Performance

Since the script has to fetch _every single block in range_ to scan for transactions initiated by a certain set of EOAs, the application typically runs slowly. To remedy this, there is an auxiliary script called `sync` (`src/sync.ts`) which takes a chain ID and start block number. The sync script will run continuously to ensure that the local blocks and transaction receipts databases are up to date, starting at the specified start block. It is recommended to keep this script running on whatever machine will be calculating the incentives every week.

The sync script may need to be terminated before running the main application. Afterward, the sync script can be restarted from the same start block, or the `db` directory can be deleted and the sync can be restarted from a new starting point. The latter strategy is useful when the `db` directory gets very large.

```bash
yarn sync <chainId> <startBlock>
```

## Running with Docker and Makefile

## Prerequisites

- Docker installed on your system
- Make installed on your system

## Quick Start

1. Build the Docker image:

```bash
make build
```

2. Start an interactive development session:

```bash
make up
```

Once inside the container, you need to run `yarn` once to install dependencies.
