# Security Model


## The entities

This packages conceptualizes the basic entities of the security model of
our system:

* **Principal** is someone who can be authenticated (by, e.g., a password)
* **Person** is someone who bears a natural identity (with, e.g., a name, or birthdate)
* **Privileges** is what is allowed to be done
* **Organisation** is a body of persons, formed for a self-given purpose
* **Role** is what a person plays in an organisation
* **AuthenticationMean** is anything a principal can be authenticated by

![classes.svg](graphviz/classes.svg)

We distinguish between `Principal` and `Person`, because in our system persons
can be both parents and children, where one or more parents may authenticate
on behalf of one or more children. Principals are for placing security concerns,
persons are for placing domain concerns.

To better deal with that split in practice, another entity is defined:

* **User** is the combination of both a principal and a person


## Deploy a snapshot update

We use 
[Sonatype OSSRH](https://central.sonatype.org/pages/ossrh-guide.html)
for hosting, the `~/.m2/settings.xml` should be setup accordingly:

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
        https://maven.apache.org/xsd/settings-1.0.0.xsd">

  <profiles>
    <profile>
      <id>sonatype-oss-release</id>
      <properties>
        <gpg.keyname>********</gpg.keyname>
        <gpg.defaultKeyring>true</gpg.defaultKeyring>
        <gpg.useagent>true</gpg.useagent>
        <gpg.lockMode>never</gpg.lockMode>
      </properties>
    </profile>
  </profiles>

  <servers>

    <server>
      <id>sonatype-staging</id>
      <username>********</username>
      <password>********</password>
    </server>

    <server>
      <id>sonatype-snapshot</id>
      <username>********</username>
      <password>********</password>
    </server>

  </servers>

</settings>
```

When this is in place and valid, `mvn deploy` will update the current snapshot artefact.


## Release a new version

### Local Maven Procedure

We use the
[Maven Release Plugin](https://maven.apache.org/maven-release/maven-release-plugin/)
for packaging and
[Sonatype OSSRH](https://central.sonatype.org/pages/ossrh-guide.html)
for hosting.

The relese plugin requries mostly three steps
(see its documentation for further details):

- `mvn release:clean`
- `mvn release:prepare`
- `mvn release:perform`

### OSSRH Staging Procedure

After performing the release, which puts the new artefact into an automatically
created staging repository, navigate to
`https://oss.sonatype.org/#stagingRepositories`. There you can complete (or fail)
the final release.


## A word on relations and generic typing

In this basic model, relations are specified in one direction only,
avoiding circular references. Implementations may introduce backward
references (OR mappers may support these conveniently).

Because of this one-way linking, not all relations are fully
expressed. For example, while the model clearly shows that a
principal may relate to a number of persons, it is not visible
that a person may legally have more than one principal.

Relations are expressed by getters on the entities. To relieve
the implementors and users of this model from tedious type-casting,
generic parameters have been specified for entities with getters.
And since ultimately all entities of this model are linked to each
other, each generic entity requires the complete parameter list,
which may look a bit scarry at first. To mitigate the scare factor,
two measures where taken:

* the generic parameter list is copy-paste, so it looks the same
on each entity
* there is an example implementation in the test folder, you may
copy from that to get started

The generic parameters appear only on the interfaces in this model.
Once you implement them in your own classes, they will disappear.
