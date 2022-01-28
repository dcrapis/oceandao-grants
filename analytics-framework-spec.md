# OceanDAO Grants Analytics Framework Spec

As the main deliverable for our R12 grant from OceanDAO, this document lays out a specification for the data infrastructure we intend to build. The main sections correspond to the main deliverables specified in the proposal.

# 1 KPI metrics and their calculation logic

We want to propose a better data schema that will make it easy to calculate KPI metrics for OceanDAO governance, run ad-hoc analyses, and add new metrics. Initially this table can simply be a csv or Airtable with required columns, in the future it can be migrated to a more scalable database.

In this section, we introduce a set of KPIs for OceanDAO, with the dual purpose of: (1) defining a set of standard metrics to look at performance over the grant rounds; (2) give a concrete example of how a dashboard enabled by such table and available to the entire Ocean Community and to the public could look like. This dashboard is currently [built](https://github.com/dcrapis/oceandao-grants/blob/main/oceandao-kpis.ipynb) using historical data that was previously [collected](https://github.com/andrewpenland/rewardsresearch/tree/main/OceanDAO/source-data) and [processed](https://github.com/andrewpenland/rewardsresearch/blob/main/OceanDAO/source-data/OceanDAO-data-cleaning-with-timestamps.ipynb) by Token Engineering Academy researchers.

In the future, the new table we want to build will allow to have the KPI dashboard updated in real time after every round. We think that this could be part of a new Analytics tab on Ocean Pearl and will follow up with their team for a collaboration.

**Last round view**

<img src="https://github.com/m3-labs/oceandao-grants/blob/main/images/Screen_Shot_2022-01-13_at_11.56.42_AM.png">

**Previous rounds view**

<img src="https://github.com/m3-labs/oceandao-grants/blob/main/images/Screen_Shot_2022-01-13_at_12.01.42_PM.png">

## KPIs by round

We list the main KPIs by round and their calculation logic. This was the result of research looking at past grant rounds performance and we have divided metrics into broad categories for *voter engagement* and for *project engagement > funding > outcome*. This is only a small example of what more granular data will enable, as an additional example in the dashboard above we compute rates that allow to immediately grasp ecosystem health, e.g., the *returning rate* of projects has been steadily increasing and the *completed rate* has been consistently above 50% starting in round 4.

**Voter engagement**

- Unique addresses
- Votes

**Project engagement**

- Projects submitted
    - New projects
    - Returning projects

**Project funding**

- Projects funded
- Amount funded

**Project outcome**

- Projects completed

## **Ad-hoc analyses**

The new table also allows to conduct ad-hoc analyses to inspect the vote distribution, the impact of whale voters on voting outcome and voting patterns of selected groups, and also deeper analysis like sybil resistance of the system versus counterfactual scenarios (e.g., with or without Quadratic Voting). We conducted some of these analyses on historical data and can be found as an example [here](https://github.com/dcrapis/oceandao-grants/blob/main/oceandao-adhoc-analysis.ipynb). We report two main highlights as example:

1. Top 5% of voters casted 25% of the total vote. This is due to both **unequal token distribution** and the fact that **before R11 voters could spend their tokens multiple times**. Note: this has since partially been addressed by implementing *simple ballots* and *quadratic voting*.
2. Most non-whale voters are always voting YES on projects, while whale votes are more differentiated. A few whales always vote NO. See figure below.

<p align="center">
<img src="https://github.com/m3-labs/oceandao-grants/blob/main/images/Screen_Shot_2022-01-28_at_1.32.29_PM.png">
</p>

> Fraction of YES votes (1.0 is 100% YES) out of total votes cast in all previous rounds for whale (True) and non-whale (False) voters. A whale is defined as an address that is in the top 5% of the voting power distribution (X00,000 of OCEAN voting power).
> 

# 2 Current DAO-data schema

In this section we map the current “data schema” that is computed by DAOBot after every round of funding. In particular we look at current input data, processing, and output tables that are available today through csv or airtable.

## 2.1 Current DAOBot workflow

There is a few steps one can follow to setup the workflow. We report them at a high level since they are already documented in the [DAOBot repo](https://github.com/oceanprotocol/DAOBot).

You need to setup API access for **Airtable** and the **Snapshot** instance for OceanDAO:

- Airtable is used to fetch all the active *proposals* in the current funding round
- Snapshot is called to update the total *votes* for each of the active proposals
    - At the end of the round the votes are updated to the finalized votes
- The data is dumped to an Airtable (or a Gsheet) with the voting results

Config

- Requires users to have Airtable and Infura API keys in the project directory as fined in the  `.env` configuration file

## 2.2 Current output data

There are two main sources of data as mentioned above, we paste here a view with a few lines and a few representative columns for each table.

1. Airtable with active proposals. Contains data at the round, proposal level of granularity with a sum for the total number of votes for each proposal.

| ipfsHash | Project Name | Yes | No | Num Voters | Sum Votes |
| --- | --- | --- | --- | --- | --- |
| QmcxWod9dGW6De8gobXErdVQNYYR7F9yZ5BKhcJGWuYnFv | DIAM | 128423.4805 | 361.8052174 | 25 | 128785.2857 |
| QmcxWod9dGW6De8gobXErdVQNYYR7F9yZ5BKhcJGWuYnFv | Athena Data Brokerage Project | 105281.0127 | 18.96153729 | 22 | 105299.9742 |
| QmcxWod9dGW6De8gobXErdVQNYYR7F9yZ5BKhcJGWuYnFv | DataBounty Platform | 35898.99679 | 418.1719327 | 13 | 36317.16873 |
1. Snapshot votes for each proposal and wallet address.

| address | choice | created | balace |
| --- | --- | --- | --- |
| 0x80083f2B6D9A748b44eD58D6374414934dcA600d | 1 | 1633994563 | 13701.4406 |
| 0x423325991513d405cdD0bc2D0634722843D29728 | 2 | 1633984691 | 1500000 |

# 3 New DAO-data schema and ETL

## 3.1 Input data required

We use the same sources of data that are currently used by DAOBot: Airtable for proposals and Snapshot for votes. The main problem currently is that this data is dumped in csv files with no structure, and the structure may change from round to round. As a result, it is hard to explore or build analyses and dashboards upon it. We propose a framework that makes these easy.

## 3.2 ETL workflow and tables

We propose a simple framework with two base tables for **Proposals** and **Votes**. Plus, a new **address-level** table or view derived from these two.

<p align="center">
<img src="https://github.com/m3-labs/oceandao-grants/blob/main/images/Screen_Shot_2022-01-28_at_12.31.47_PM.png">
</p>

> Entity relationship diagram for base tables.
> 

### 3.2.1 New address-level data table

The new address level data table can be easily computed from the two main tables above. We could set up a simple ETL for this, but provided that the two main tables are maintained and updated in the right format a simple merge would do. The address-level table schema is similar to the one we used in the demonstrations and analyses in Section 1. See diagram below.

<p align="center">
<img src="https://github.com/m3-labs/oceandao-grants/blob/main/images/Screen_Shot_2022-01-28_at_12.31.54_PM.png">
</p>

This table will have the required fields listed below plus some optional fields that can be added in the future if/when the voting process is modified.

- **Address: type `address` 20-byte value, not null.**
- **Balance: type `uint`, not null.**
- **Vote: type `uint`, not null.**
- **TimeStamp**: **type** `unit`, **not null**.
- **Round: type `uint`, not null.**
- **Project Name: type `string`, not null.**
- **Proposal State: type `string`, not null.**
- **Proposal Standing: type `string`, not null.**
- **Grant Category: type `string`, not null.**
- **Earmarks: type `string`, not null.**
- **OCEAN Granted: `bool`type, not null.**

### 3.3.2 How to add new fields

There are modifications that can enhance the current DAOBot process which in turn significantly improves data analyses:

- Airtable could be modified by adding new address level tables as described in section 3.3.1.
- The current DAOBot’s API remains the same, however, it requires new back-end component to store/load into/from the new table.

# Resources

[Ocean Port Proposal R12](https://port.oceanprotocol.com/t/davide-crapis-oceandao-grants-analytics-framework/1159)

[Output spreadsheet](https://docs.google.com/spreadsheets/d/1e4xb6m-aKcBhwob_p7ereSFneXFPpDjUbSjz13smmHI/edit#gid=626352412)

[TEA Research Processing of Historical Data](https://github.com/andrewpenland/rewardsresearch/blob/main/OceanDAO/source-data/OceanDAO-data-cleaning-with-timestamps.ipynb)

[Example output for ad-hoc analysis](https://docs.google.com/spreadsheets/d/1L0DszSfkb_hx1WKRcbeenklJdK78G9Gx9TAsjVa3ZF8/edit#gid=0)
