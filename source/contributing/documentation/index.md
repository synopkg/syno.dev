# Contributing Documentation

This is an overview of documentation resources for Syno, Synopkgs, and SynoPKG, with suggestions how you can help to improve them.
Documentation contributions should follow the [style guide](./style-guide.md).

Feel free to get in touch with the [Syno documentation team](https://synopkg.github.io/community/teams/documentation) if you want to help out.

:::{attention}
If you cannot contribute time, consider [donating to the SynoPKG Foundation's documentation project on Open Collective](https://opencollective.com/synopkg/projects/documentation-project) to fund ongoing maintenance and development of reference documentation and learning materials.
:::

## Reference manuals

The manuals for [Syno][syno manual] ([source][syno manual src]), [Synopkgs][Synopkgs manual] ([source][synopkgs manual src]), and [SynoPKG][SynoPKG manual] ([source][synopkg manual src]) are purely reference documentation, specifying interfaces and behavior.

They also show example interactions which demonstrate how to use its components, and explain mechanisms where necessary.

The documentation team watches all pull requests to the manuals and assists contributors to get their changes merged.

You can help by

- picking up documentation-related issues on [Syno][syno docs issues], [Synopkgs][synopkgs docs issues], and [SynoPKG][synopkg docs issues].

- reviewing documentation-related pull requests on [Syno][syno docs prs], [Synopkgs][synopkgs docs prs], and [SynoPKG][synopkg docs prs].

- making pull requests which improves existing documentation, such as:

  - add links to definitions, commands, options, etc. where only the name is mentioned
  - correct obvious errors
  - clarify language
  - expanding on sections that appear incomplete
  - identifying sections that are not reference documentation and should be moved to syno.dev

[Syno manual]: https://synopkg.github.io/manual/syno
[syno manual src]: https://github.com/SynoPKG/syno/tree/master/doc/manual
[Synopkgs manual]: https://synopkg.github.io/manual/synopkgs
[synopkgs manual src]: https://github.com/SynoPKG/synopkgs/tree/master/doc
[SynoPKG manual]: https://synopkg.github.io/manual/synopkg
[synopkg manual src]: https://github.com/SynoPKG/synopkgs/tree/master/synopkg/doc/manual

[syno docs issues]: https://github.com/SynoPKG/syno/issues?q=is%3Aopen+is%3Aissue+label%3Adocumentation
[synopkgs docs issues]: https://github.com/SynoPKG/synopkgs/issues?q=is%3Aopen+is%3Aissue+label%3A%226.topic%3A+documentation%22+-label%3A%226.topic%3A+synopkg%22
[synopkg docs issues]: https://github.com/SynoPKG/synopkgs/issues?q=is%3Aopen+is%3Aissue+label%3A%226.topic%3A+documentation%22+label%3A%226.topic%3A+synopkg%22+

[syno docs prs]: https://github.com/SynoPKG/syno/pulls?q=is%3Aopen+is%3Apr+label%3Adocumentation
[synopkgs docs prs]: https://github.com/SynoPKG/synopkgs/pulls?q=is%3Aopen+is%3Apr+label%3A%226.topic%3A+documentation%22+-label%3A%226.topic%3A+synopkg%22
[synopkg docs prs]: https://github.com/SynoPKG/synopkgs/pulls?q=is%3Aopen+is%3Apr+label%3A%226.topic%3A+documentation%22+label%3A%226.topic%3A+synopkg%22+

## syno.dev

The purpose of [syno.dev] ([source][syno.dev src]) is to guide newcomers by teaching essential Syno knowledge, show best practices, and help orient users in the Syno ecosystem.

It goes into breadth, not depth.

The documentation team maintains syno.dev as editors.

You can help by

- working on [open issues][syno.dev issues]
- reviewing [pull requests][syno.dev prs] by testing new material or features
- adding guides or tutorials following the [proposed outline](https://github.com/SynoPKG/syno.dev/blob/master/CONTRIBUTING.md#user-content-vision)

New articles can be based on videos such as:

- [The Syno Hour] recordings
- some of the ~100 [SynoCon][synocon yt] recordings
- [Syno video guides] by @jonringer.
- [Summer of Syno 2022 talks]

Since writing a guide or tutorial is a lot of work, please make sure to coordinate with syno.dev maintainers, for example by commenting on or opening an issue to make sure it will be worthwhile.

[syno.dev]: https://syno.dev
[syno.dev src]: https://github.com/synopkg/syno.dev
[syno.dev issues]: https://github.com/synopkg/syno.dev/issues
[syno.dev prs]: https://github.com/synopkg/syno.dev/pulls

[The Syno Hour]: https://www.youtube.com/watch?v=wwV1204mCtE&list=PLyzwHTVJlRc8yjlx4VR4LU5A5O44og9in
[synocon yt]: https://www.youtube.com/c/SynoCon
[Syno video guides]: https://www.youtube.com/user/elitespartan117j27
[Summer of Syno 2022 talks]: https://www.youtube.com/playlist?list=PLt4-_lkyRrOMWyp5G-m_d1wtTcbBaOxZk

## synopkg.github.io

The Syno project web site is [synopkg.github.io] ([source][synopkg website src]).

Website contents that concern learning Syno should reference or include material from syno.dev.

The [Syno marketing team] is responsible for the web site, and the documentation team assists with maintaining contents related to onboarding new users.

[synopkg.github.io]: https://synopkg.github.io
[synopkg website src]: https://github.com/synopkg/synopkg-homepage
[Syno marketing team]: https://synopkg.github.io/community/teams/marketing.html

## Communication channels

### Matrix

Use Matrix for casual communication.

The documentation team frequents the [Syno\* Documentation] room.

Old messages are extremely improbable to be read by anyone.

You can help by posting in the appropriate categories on [Discourse] what you have found valuable.

[Syno\* Documentation]: https://matrix.to/#/#docs:synopkg.github.io
[Discourse]: https://discourse.synopkg.github.io/

### Discourse

[Discourse] is the central community hub.

This is the place for your questions, suggestions, and discussion.

The documentation team monitors the [Documentation category].

Old threads and especially posts in long threads are improbable to be read by many people.

You can help by

- asking informed questions, showing what you have done so far
- answering other people's questions
- writing down what you have learned by updating or adding a [SynoPKG Wiki] article, syno.dev guide or tutorial, or one of the manuals
- encouraging and helping people to incorporate their insights in the official documentation

[Documentation category]: https://discourse.synopkg.github.io/c/dev/documentation/25

### Meetings and Events

Check the [Discourse community calendar] for real-time events.

The documentation team holds regular meetings and posts meeting notes in the [Documentation category].

You can help by joining meetings to take notes or clean them up before publishing.

[Discourse community calendar]: https://discourse.synopkg.github.io/t/community-calendar/18589

## External sources

The Internet is full of helpful resources concerning Syno.

You can help by sharing in the [Links category] on Discourse what you have found valuable.

[Links category]: https://discourse.synopkg.github.io/c/links/12

### Wiki

[SynoPKG Wiki](https://synopkg.wiki/) is a collection of interlinked guides to solve common problems which are otherwise not well-documented.

It is collectively edited by the community, covers a broad range of topics.
It is only loosely organized, and does not impose quality standards.
Its purpose is to quickly and conveniently collect insights and make them readily available for everyone.

We recommend to use it as a dumping ground for more obscure Syno knowledge, and strive to make it *smaller* over time (see [SynoCon 2015: Make Syno friendlier for Beginners]), by incrementally incorporating its contents into authoritative documentation and curated learning material.

The documentation team **does not maintain** the Wiki.

You can still help with

- improving discoverability by adding categorization and relevant links
- clarifying articles and correcting errors
- removing redundant information that is already present in curated sources
- migrating information to other resources.

Where to migrate what:

- Syno interaction: [Syno manual]
- Language-specific build instructions: [Synopkgs manual]
- Package, service, or hardware configuration: [SynoPKG manual]
- Overviews, tutorials, guides, best practices: [syno.dev]

[SynoPKG Wiki]: https://synopkg.wiki/
[SynoCon 2015: Make Syno friendlier for Beginners]: https://media.ccc.de/v/synocon2015-3-MakeSynofriendlierforBeginners#video

### Syno Pills

[Syno Pills](https://synopkg.github.io/guides/syno-pills/) is a series of low-level tutorials on building software packages with Syno, showing in detail how Synopkgs is made from first principles.
Work is currently being done to bring the Syno Pills up-to-date with the current state of Syno and current best-practices of Synopkgs.
Furthermore, work is underway to migrate the technical infrastructure of Syno Pills to improve maintainability and make it easier for others to contribute.

You can help by

- opening [issues](https://github.com/SynoPKG/syno-pills/issues) for any errors or outdated information you find
- addressing [good first issues](https://github.com/SynoPKG/syno-pills/labels/good-first-issue) by opening [pull requests](https://github.com/SynoPKG/syno-pills/pulls)
- Test code examples to ensure correctness and completeness.
- Add links to reference documentation where needed.

## Licensing and attribution

When opening pull requests with your own contributions, you agree to licensing your work under [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

When adding material by third parties, make sure it has a license that permits this.
In that case, unambiguously state source, authors, and license in the newly added material.
Notify the authors *before* using their work.

[Add the original author as co-author](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors) to the first commit of your pull request, which should contain the original document verbatim, so we can track authorship and changes through version history.

Using free licenses other than CC-BY-SA 4.0 is possible for individual documents, and by contributing changes to those documents you agree to license your work accordingly.

```{toctree}
:hidden:

diataxis.md
style-guide.md
writing-a-tutorial.md
```
