# Governance

[WP Rig](https://github.com/wprig/wprig/) is an open source project that depends on contributions from the community. As long as they abide by the project’s [Code of Conduct](), anyone may contribute to the project at any time by submitting code, participating in discussions, making suggestions, or any other contribution they see fit. This document describes how various types of contributors work within the WP Rig project and how decisions are made.

## Roles and Responsibilities

### Community members
_Everyone_ who is involved in any form with the project must abide by the project’s [Code of Conduct](). Everyone is expected to be respectful of fellow community members and to work collaboratively respective of the Code of Conduct. Consequences for not adhering to these Guidelines are listed in their respective documents.

### Users
Users are community members who have a need for the project. They are typically developers building WordPress themes. Anyone can be a User; there are no special requirements and WP Rig is licensed under [GPL v3.0](https://github.com/wprig/wprig/blob/master/LICENSE). Common User contributions include using WP Rig to build themes, evangelizing the project (e.g., display a link on a website and raise awareness through word-of-mouth), informing developers of strengths and weaknesses from a new user perspective, or providing moral support (a “thank you” goes a long way).

Users who continue to engage with the project and its community will often become more and more involved. Such Users may find themselves becoming [Contributors](#Contributors), as described in the next section.

### Contributors
Contributors are community members who contribute in concrete ways to the project, most often in the form of code and/or documentation submitted through [Pull Requests](https://help.github.com/en/articles/about-pull-requests) (PRs) to the [`develop` branch of the WP Rig GitHub repository](https://github.com/wprig/wprig/tree/develop). Anyone can become a Contributor, and contributions can take many forms. There is no expectation of commitment to the project, no specific skill requirements, and no selection process. Contributors must follow the [Code of Conduct]().

Contributors:
- Have read-only access to source code and therefore can submit changes via pull requests.
- Have their contribution reviewed and merged by a [Maintainer](#Maintainers) or [Owner](#Owners). Owners and Maintainers work with Contributors to review their code and prepare it for merging.
- May also comment on issues and pull requests. While their approval or disapproval is not decisive, it is a good way to provide feedback to those making decisions.

As Contributors gain experience and familiarity with the project, their profile within, and commitment to, the community will increase. At some stage, they may find themselves being nominated for becoming a Maintainer by an existing Maintainer or Owner.

### Maintainers
Maintainers are community members who have shown that they are committed to the continued development of the project through ongoing engagement with the community. Maintainers are given push/write access to the project’s GitHub repos.

Maintainers:

- Are expected to work on public branches of their forks and submit pull requests to the `development` branch.
- Must submit pull requests for all their changes.
- May label and close issues.
- May only merge other people's pull requests once the PR has been approved by two other Maintainers.
- Should ask for additional review from other Maintainers or Owners on other people's PRs that are disruptive or controversial.

To become a Maintainer one must:

- Have shown a willingness and ability to participate in the project in a helpful and collaborative way with the WordPress community.
- Typically, a potential Maintainer will need to show that they have an understanding of and alignment with the project, its objectives, and its strategy.
- Have contributed a significant amount of work to the project (e.g. in the form of PRs or PR reviews), thereby demonstrating their trustworthiness and commitment to the project.
- Read and agree to abide by the [Code of Conduct]().
- File an issue in the [wprig/wprig](https://github.com/wprig/wprig) repository and record “I have read, and agree to abide by, the [WP Rig Code of Conduct].”

New Maintainers can be nominated by any existing Maintainers. Once they have been nominated, there will be a vote by the Maintainers.

It is important to recognize that being a Maintainer is a privilege, not a right. That privilege must be earned and once earned it can be removed by a majority vote by the Maintainers. However, under normal circumstances the Maintainer status exists for as long as the Maintainer wishes to continue engaging with the project. Inactive Maintainers (no activity on the project for longer for 4 months or more) might be marked as inactive or removed after a majority vote of Maintainers and may re-enter when they choose to contribute again.

#### List of current peers
- Andrew Taylor (@ataylorme)
- Benoit Chantre (@benoitchantre)
- Felix Arntz (@felixarntz)
- Morten Rand-Hendriksen (@mor10)
- Rachel Cherry (@bamadesigner)

A Maintainer who shows an above-average level of contribution to the project, particularly with respect to its strategic direction and long-term health, may be nominated to become an Owner, described below.

### Owners
The WP Rig project is governed by the [Owner group](#list-of-current-owners). They are collectively responsible for high-level guidance of the project.

The [Owner group](#list-of-current-owners) has final authority over this project including:

- Technical direction of the project, especially infrastructure PRs and linting and/or schema decisions.
- Project governance and process (including this policy and any updates).
- Contribution policy.
- GitHub repository hosting.

Being an Owner is not time-limited. There is no fixed size of the Owner group. The Owner group should be of such a size as to ensure adequate coverage of important areas of expertise balanced with the ability to make decisions efficiently.
An Owner may be removed from the Owner group by voluntary resignation, or by a standard Owner group motion, including for violations of Code of Conduct.

Changes to the Owner group should be posted in the agenda, and may be suggested as any other agenda item (see [Project Meetings](#project-meetings) below).

Owners fulfill all requirements of Maintainers, and also:

- Ensure the smooth running of the project.
- Approve changes to this document.
- Manage and merge commits to the `master` branch.

To become an Owner one must fulfill at least the following conditions and commit to being a part of the community for the long-term.

- Have worked in a helpful and collaborative way with the WordPress and WP Rig communities.
- Have given good feedback on others’ submissions and displayed an overall understanding of the code quality standards for the project.
- Have the ability to drive the project forward, manage requirements from users, and taking on responsibility for the overall health of the project.

An individual is invited to become an Owner by existing Owners. A nomination will result in discussion and then a decision by the Owner group.

#### List of current Owners
- Morten Rand-Hendriksen (@mor10)
- Rachel Cherry (@bamadesigner)

## Decision Making
Decision making generally follows a [Consensus-seeking decision-making](https://en.wikipedia.org/wiki/Consensus-seeking_decision-making) model.

When an agenda item has appeared to reach a consensus, the moderator will ask “Does anyone object?” as a final call for dissent from the consensus.

If an agenda item cannot reach a consensus, an owner can call for either a closing vote or a vote to table the issue to the next meeting. The call for a vote must be approved by a majority of the owners or else the discussion will continue. Simple majority wins.

## Project Meetings
There are bi-weekly project meetings; they are scheduled in a [shared calendar](https://calendar.google.com/calendar/embed?src=wprigio%40gmail.com&ctz=America%2FChicago), and using tools that enable participation by the community. 

## Credits
This work is a derivative of [MDN Browser Compat Data GOVERNANCE](https://github.com/mdn/browser-compat-data/blob/master/GOVERNANCE.md), a derivative of [ESLint Governance](https://github.com/eslint/eslint.github.io/blob/master/docs/maintainer-guide/governance.md), [YUI Contributor Model](https://github.com/yui/yui3/wiki/Contributor-Model), and the [JS Foundation TAC Charter](https://github.com/JSFoundation/TAC).
