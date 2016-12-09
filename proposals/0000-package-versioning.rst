.. proposal-number:: Leave blank. This will be filled in when the proposal is
                     accepted.

.. highlight:: haskell



Managing Version Constraints
============================

As the Haskell ecosystem matures, there are starting to be multiple mechanisms
to manage sets of packages that are known to be compatible with each other.

Currently there is the original one, cabal-install/hackage, as well as
stack/stackage. This set is bound to grow in future.

Each of these takes a different approach

**cabal-install/hackage**

This relies on a constraint solver, using the dependency constraints captured in
the individual package cabal files.

In order to make this problem tractable it requires the `PVP <http://pvp.haskell.org/>`_

This has the following requirement

  When publishing a Cabal package, you SHALL ensure that your dependencies in
  the build-depends field are accurate. This means specifying not only lower
  bounds, but also upper bounds on every dependency.

There is a site that checks whether the solver is able to find a solution for
each package, at http://matrix.hackage.haskell.org/

There is also a requirement for specifying as narrow as possible an upper bound
for a given dependency which means that if the PVP is followed it should
require the first 2 digits of the version number to stay the same as the
currently used one. It is likely that a wider range may work in future, but
until the new version arrives with a different set of numbers, this cannot be
assumed.

In this document version constraints derived from this requirement will be
called *implicit constraints*.

**stack/stackage**

This ecosystem works by constructing a series of snapshots, each of which
contains a set of packages that are curated to be able to build together and
interoperate.

There is a stable series, and a series of named nightlies that capture the work
in progress toward the next stable release.

The `stack` tool uses a snapshot to exactly specify a set of packages, so no
version bounds are required in a cabal file at all.

If packages outside a snapshot are needed for a project built by stack, the
exact version numbers must be specified in the `stack.yaml` config file for the
project.

From Michael Snoyman


    The existence of snapshots ala Stackage is a requirement for many people in
    the community, and no amount of dep solving will get rid of it. Similarly,
    some people take dep solving as a requirement. We need to acknowledge that
    both solutions will endure.

    Conflating speculative and known bounds has been the major source of tension
    in the versioning debate

    Hackage revisions are far too powerful a tool to solve a very simple
    problem; mutable bounds information should be stored separately. Personally:
    I think the cabal file should only contain known version bounds, and
    speculative version bounds should be kept separately, but there's no reason
    why someone couldn't put speculative version bounds in a .cabal file going
    forward. We simply shouldn't _require_ speculative bounds to be present.

    We took a very pragmatic approach to Stackage, which was to acknowledge what
    package authors were most likely to be able to comply with, and tailored our
    approach to that. Any version bounds discussion must have the same
    philosophy if it is to succeed: we need to acknowledge what authors have
    been willing to do, and what we can reasonably educate people on

    Authors make mistakes with the PVP (I had two package failures in the past
    week because of it, even though I was blamed for one of them falsely).
    Automated tooling will make mistakes with bounds. The PVP itself - even if
    fully followed - does not guarantee 100% success. The only solution that
    guarantees a 100% success of a build plan is curation (and even that is
    brittle due to, e.g., different OSes). So we cannot reject a solution
    because it won't work in some corner case. We need to minimize the
    probability of failure, accepting that failure will inevitably happen.


[1] https://gist.github.com/snoyberg/f6f10cdbea4b9e22d1b83e490ec59a10


Motivation
----------

The problem addressed in this proposal is to reduce the burden on package
maintainers who are currently required to update the implicit constraints when
it becomes evident that subsequently published versions of a given dependency
are indeed compatible with the current package.

Failing to do so can cause the hackage solver to fail, resulting in not being
able to use particular packages.

Since the problemmatic constraints are essentially outside the control of the
original developer, some way of moving the management of these out of the hands
of the package maintainer is proposed.

In Michael Snoyman's words

* Let people continue adding in weird exceptions and cases that they know for
  certain

* Automate the thing most people are doing, and thereby ensure (1) even people
  who don't care about the PVP are providing that info, (2) lower bound bumps
  are not forgotten either, and (3) it's easier to test upper bound relaxes
  going forward through automated tooling


Proposed Change
---------------

The details of this section will be fleshed out as part of a discussion/dialogue
with the interested parties.

One possible solution is put forward
`here <https://github.com/haskell/cabal/issues/3729>`_, but it probably
requires too large a change to be practicable.

Other solutions involve automatically setting the required implicit constraints
on upload, using the `--pvp-bounds=both` option when uploading via stack, and a
similar (to be added) feature for cabal.

Or the implicit bounds could be calculated on hackage as part of the upload
process, so that maintainers are then only required to specify constraints that
have to be satisfied, failing which the package cannot be built.

Other solutions ...

The devil is in the details with each of these strategies. We should now being
constructing concrete proposals around the alternatives, on the way to coming up
with a single solution that works for all.

I define works for all as

* hackage/cabal solver is able to come up with a build plan, and its strategy
  can be improved over time.
* Package maintainers are not subject to needless busywork. Most are volunteers,
  their time is precious and is better spend building things for us all.
* It does not impede the workings of stack/stackage.

Ideally, once this problem is resolved, part of the improvement of safely
updating the implicit bounds will be feedback from the stackage builder.

Drawbacks
---------

What are the reasons for *not* adopting the proposed change. These might include
complicating the language grammar, poor interactions with other features, 

Alternatives
------------

Here is where you can describe possible variants to the approach described in
the Proposed Change section.

Unresolved Questions
--------------------

Are there any parts of the design that are still unclear? Hopefully this section
will be empty by the time the proposal is brought up for a final decision.
