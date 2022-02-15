# Remove Description in Hash Generation

- **Status:** proposed
- **Decisores:** [wiliansilvazup](https://github.com/wiliansilvazup), [nathanmartinszup](https://github.com/nathanmartinszup), [matheusalcantarazup](https://github.com/matheusalcantarazup), [iancardosozup](https://github.com/iancardosozup)
- **Date:** 2022-02-15
- **Tags:** technology, rules, semantic, security, engine, sast, horusec

## Context and problem

Horusec is receiving several contributions in the project, aiming at improving the project today we need to mitigate the risks if we keep the description in the generation of the hash so when an improvement in the description of the vulnerability can be changed without worrying about the risks of "Breaking Changes".

## Decision

We will remove the field `Description` in the generation of the hashes and we will create a new field named `OldHashes` in the export of the vulnerability so that the external tools have an option to treat any of the old or new hashes.

### You will need to change the Devkit to have this new component of "OldHashes"
In the project [Horusec-Devkit](https://github.com/ZupIT/horusec-devkit) We have the [struct vulnerability](https://github.com/ZupIT/horusec-devkit/blob/main/pkg/entities/vulnerability/vulnerability.go#L28) In it you need to create a new field for example:
```go
type Vulnerability struct {
  ...
  OldHashes pq.StringArray `json:"oldHashes" gorm:"type:text[]"`
  ...
}
```

### You will need to communicate this change to the community from which we have decided to follow
Communication can be done in all Horusec development processes:
- Development: [Coverage for ADR](https://github.com/ZupIT/horusec/issues/990)
- BugHunting: [validation of the issue](https://github.com/ZupIT/horusec/issues/990)
- Beta: Adding on our Release Notes and [Updating issue](https://github.com/ZupIT/horusec/issues/990)
- Release Candidate: Adding on our Release Notes and [Updating issue](https://github.com/ZupIT/horusec/issues/990)

### You will need to change the Horusec-Platform to support the old hash fields and new hash
In Horusec-Platform we should validate when you receive this `OldHashes` field and check if it already exists in our database.
If there is any hash, then we must use what was already registered for the current analysis.
If there is no hash, then we must register this new.

### You will need to add a warning on Horusec-cli
In Horusec-CLI we must check if any vulnerability is using both old and new hashes.
If the user is exporting his analysis to another different source of the text type (JSON, SONARCHBE, SARIF, HORUSEC-API) We must add a warning at the output of the application demonstrating to the user who has changed and he should be aware of the changes.

### After switching to a new format you should wait for at least three minors or six-month versions to change again
We do not want to keep code that does not make sense for the project as this is a "Workaround" to be transparent with the community and that we are concerned about user experience in the project.
To be able to remove the `OldHashes` field to wait at least three minors versions or 6 months.

## Options considered

### Just remove the description of the hash generation and let break in exports
#### Pros
  - In this scenario it would be simpler the changes and the impact on the well lower code.
#### Cons.
  - User experience would be pretty bad if there is an application using the project with various false positive treatments and accepted risks.

### Keep the old format statically
#### Pros
  - It would also be interesting to create a static `OldDescriptions` file and add it to the project so we can generate the hash based on it.
#### Cons.
  - The maintenance of this would be quite large and we are just pushing the problem for a short-term period.

### Leave the field Description and create a new field "NewDescriptions"
#### Pros
  - As well as removing the description in this format it would also be interesting and the impact on the well lower code.
#### Cons.
  - But we fall into the same scenario where maintenance would be very large and totally ignored the user experience of the application.

## Links