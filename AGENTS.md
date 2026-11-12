# AGENTS.md: smart-contract-api

Instructions for coding agents working in this repository. Read this before changing anything.

## What this is

The Java API that CREDITS smart contracts are written against: the base classes a contract extends
(`SmartContract` v0 to v3), the token standards and the annotations. The classes here are compile-time
stubs (methods return `null` or `0`); the contract executor supplies their behaviour at run time.

Related repositories of [CreditsCommunity](https://github.com/CreditsCommunity):

- `contract-executor`: builds against `sc-api-internal` 2.2 from this repository and ships the
  runtime side as `sc-api-support.jar` (its branch `build/reproducible-1518`).
- `node`: runs the executor; its `samples/contracts/` are contracts written against this API.
- `thrift-api-client`: the other Java dependency of the executor.

## Branches

`master`, `dev` and `build/reproducible-1518` point to the same commit (`677c567f`, the commit
contract-executor build 1518 is built from). `mvn-repo` holds Maven artifacts published by the
original project. There is no CI in this repository: the executor's CI builds it at a pinned commit.

## Platform constraints

- Mainnet runs the official node 4.2.532 with executor build 1518, built from this commit. Contracts
  deployed on mainnet were compiled against these classes: treat every public class, method
  signature and annotation as frozen. A change here is consensus-relevant if it can change how a
  contract compiles or what it computes, and must never be made unilaterally.
- On mainnet a contract cannot call another contract: executor 1518 fails every
  `invokeExternalContract`, read-only calls included. No value, not even CS, passes from one contract
  to another: the node refuses a contract's `sendTransaction` to a contract. Enabling either changes
  consensus and needs a coordinated, height-gated upgrade of all trusted-capable nodes.
- `sendTransaction(String from, String to, double amount, ...)`: amounts are `double`s. A contract
  method called with arguments that do not match its declared types fails, and the fee is still paid.

## Layout

| Path | Contents |
|---|---|
| `sc-api/` | artifact `com.credits.sc-api:sc-api` 3.0, Java module `com.credits.sc.api` (its `module-info` does not export `v3`) |
| `sc-api/.../scapi/v0` ... `v3` | `SmartContract` versions (v0 and v1 deprecated); `v0` `BasicStandard`/`ExtensionStandard`, `v1` `BasicTokenStandard`/`ExtensionTokenStandard` |
| `sc-api/.../scapi/annotations` | `@Getter`, `@ContractAddress`, `@ContractMethod`, `@UsingContract(s)` |
| `sc-api-internal/` | `sc-api-internal` 2.2: the abstract `SmartContractApi` (internal v0, v1) that `SmartContract` v2 and v3 extend; exported only to the executor, `sc-api` and `sc-api-support` |
| `pom.xml` | parent `sc-api-root`, Java 11 |

## Build

Java 11 target, Maven. There are no tests.

```sh
mvn -B -DskipTests package     # sc-api/target/sc-api-3.0.jar, sc-api-internal/target/sc-api-internal-2.2.jar
mvn -B -DskipTests install     # what the executor build does before building the executor
```

For a build byte-identical to 1518, follow the executor's `BUILDING.md` (JDK 11.0.3, its Maven
settings that mirror every repository except Maven Central to vendored artifacts).

## Conventions

- One topic per pull request. Docs, comments and commit messages in English.
- Never `git add -A` or `git add .`: stage explicit paths. Never delete branches after a merge.
- A change here is tested by building contract-executor against it and running the executor's CI
  (its harness compares every contract result with build 1518).

## Pitfalls

- Do not change the version numbers in the poms: the executor resolves `sc-api-internal` 2.2 by
  version.
- `@UsingContract` does not make contract-to-contract calls work on mainnet (see above).
- `@Getter` only documents that a method is meant to be read without a transaction.

## More

- The node's `samples/contracts/README.md`: how contracts are written, deployed and called, and
  what the executor's sandbox refuses; `docs/tokens.md` for the token standards.
- contract-executor's `BUILDING.md` (branch `build/reproducible-1518`).
- Maintainers' agents: a shared Basic Memory knowledge base (project `credits`) holds platform facts,
  procedures and incident notes; use it when your environment provides it.
