# Plan for New Modules Implementation

This document outlines the plan for building a new implementation to support ECMAScript modules in Node.js. The general idea is to start with a “minimal kernel” as Phase 1, which consists of features that the @nodejs/modules group have agreed will be necessary for all potential iterations of our ESM implementation. Phase 1 does _not_ include features that preclude other potential features or implementation approaches; and Phase 1 also does not include some features that should naturally be built in a later phase of development, for example because those features depend on features planned for Phase 1. The minimal kernel/phase 1 is _not_ intended to be merged into Node core or released; it is only a starting point for gradually building layers of consensus.

At every phase, the following standards must be maintained:

* Spec compliance ([#132](https://github.com/nodejs/modules/issues/132)): We must always follow the ES spec.
* Browser equivalence ([#133](https://github.com/nodejs/modules/issues/133)): There’s room for debate in specific cases, but in general if Node is doing something that browsers also do, Node should do it in the same way. Alternatively, code that executes in both environments should produce identical results.
* Don’t break CommonJS ([#112](https://github.com/nodejs/modules/issues/112)): We cannot cause breaking changes with regards to CommonJS.

See also the [features list in the README](https://github.com/nodejs/modules#features).

## Phase 1: The Minimal Kernel

These features will be part of the first phase of development:

* `createRequireFunction` ([nodejs/node#19360](https://github.com/nodejs/node/pull/19360)) is the only way to import CommonJS into an ES module, for now.
  - `import.meta.require` does not fail early enough.
  - Hold off on `import` statements for CommonJS until more progress is made on the dynamic modules spec.

* `import` statements will only support files with an `.mjs` extension, and will import only ES modules, for now.
  - In a later phase, the intention is to move forward with format databases to map extensions and support multiple use cases.
  - No JSON or native modules; `createRequireFunction` can be used to get these.


### How will we get from where we are to Phase 1

* Remove support in the `import` statement of formats other than ESM:
  - No CommonJS.
  - No JSON.
  - No native modules.
  - Implemented in https://github.com/nodejs/ecmascript-modules/pull/3

* Remove dynamic path searching:
  - No extension adding.
  - No directory resolution, including no support for `index.js` or `index.mjs`.
  - Still support the `main` field, but it must specify a full filename.
  - Implemented in https://github.com/nodejs/ecmascript-modules/pull/2

* Add `createRequireFunction`.
  - Implemented in https://github.com/nodejs/node/pull/19360


## Features Expected in Later Phases

These features are agreed upon, but make sense to include in a later phase of development; or there is agreement on the need for this feature, but a lack of consensus on the implementation details:

* Browser-compatible specifier resolution ([#109](https://github.com/nodejs/modules/issues/109)), a.k.a. bare imports.
  - Implementation to be discussed, possibly based on [package name maps](https://github.com/domenic/package-name-maps).

* `import.meta.url`.

* Dynamic `import()`.

* User-configurable map for file extensions to parse goals, a.k.a. `mimes` field ([#160](https://github.com/nodejs/modules/pull/160)).
  - This can expand the uses of the `import` statement to import ES modules from `.js` files, as well as support JSON and other formats.
  - Common formats like JSON or WASM will have default mappings.
  - Need to work out how this functionality interacts with loaders.

* Dynamic path searching.
  - Requiring the full path is an issue for tooling and a long term solution is required.
  - Migration strategies also have issue with this.
  - Need to resolve issues related to building this in a browser-compatible way.

* Multi-mode packages ([#94](https://github.com/nodejs/modules/issues/94)).

* Loaders ([#82](https://github.com/nodejs/modules/issues/82)), ([#96](https://github.com/nodejs/modules/issues/96)).
  - Implementation to be determined.

* ESM in executable files ([#152](https://github.com/nodejs/modules/issues/152)).

* Callable resolver ([#157](https://github.com/nodejs/modules/issues/157)).

* Mock modules (injection) ([#98](https://github.com/nodejs/modules/issues/98)).


## Other Features

The features on this list cannot be said to be sure to come in a later phase, for one reason or another. It may be that there’s a dispute over whether the feature can be achieved in a spec compliant way, or a browser-compatible way. It could be that the group might decide the feature simply isn’t worth including in the final implementation, for design or other reasons. A feature’s inclusion on this list does _not_ mean that it might not move up to the “expected in later phases” list, but rather only that there are issues to be resolved before it can do so.

* Named imports from CommonJS modules via an `import` statement, e.g. `import { shuffle } from 'underscore'` ([#81](https://github.com/nodejs/modules/issues/81)):
  - Waiting on updates to the spec to allow this.

* Any CommonJS imports via an `import` statement ([#100](https://github.com/nodejs/modules/issues/100)).
  - See previous.

* Arbitrary sources for module source text ([#97](https://github.com/nodejs/modules/issues/97)).
  - We would need to work out how to make this compatible with `import` statements, the `mimes` field, etc. Perhaps implement via a loader?

* Specifier resolution customization ([#110](https://github.com/nodejs/modules/issues/110)).
  - This raises implementation concerns, such as for static analysis. Perhaps implement via a loader?

* Package encapsulation ([#111](https://github.com/nodejs/modules/issues/111)):
  - Depends on implementation of browser-compatible specifier resolution ([#109](https://github.com/nodejs/modules/issues/109)), a.k.a. bare imports.

* Conditional imports ([#113](https://github.com/nodejs/modules/issues/113)):
  - Depends on top-level `await`, which is working its way through the spec process.

* WASM modules (#[106](https://github.com/nodejs/modules/issues/106)).
  - There’s a lot of other work to do to support WASM in Node in general; the ES modules implementation can ship without support for WASM modules, so long as there’s a clear path for adding WASM support later.