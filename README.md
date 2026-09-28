# psalm-plugin-api

This package must be required by [Psalm](https://psalm.dev) plugins in order to require a specific version of the Psalm plugin API.  

This is a separate, empty metapackage: its major version will be bumped every time a breaking change occurs within Psalm's plugin API, breaking changes will be listed in this README; minors will also be bumped when adding new features to Psalm's plugin API.  

This allows breaking changes within Psalm's plugin API, without requiring a major in Psalm itself (which will be reserved only for breaking changes in the CLI API or other major behavioral changes).  

## Changelog

### Unreleased

- [BC] Purity is now tracked as a bitmask of capabilities (`Psalm\Storage\Capabilities`) instead of the four ordered levels of `Psalm\Storage\Mutations`, which was removed. Every `$allowed_mutations` property/parameter became `$capabilities` (`Psalm\Context`, `Psalm\Storage\FunctionLikeStorage`, `Psalm\Storage\ClassLikeStorage`, `Psalm\Plugin\DynamicFunctionStorage`, docblock manipulators), `MethodStorage#$containing_class_allowed_mutations` became `$containing_class_capabilities`, and `StatementsSource#signalMutation()`/`signalMutationOnlyInferred()` take the required capabilities.
- [BC] `Psalm\Type\Atomic\TCallable#$allowed_mutations` and `Psalm\Type\Atomic\TClosure#$allowed_mutations` were replaced with a `$purity` union type (a `Psalm\Type\Atomic\TCapabilities` set, or a purity template), with `getCapabilities()`/`setPurity()` helpers; the constructors take an `int|Union $purity`.
- [BC] `dictionaries/ImpureFunctionsList.php` now maps each function to its capabilities instead of listing impure functions, and `Psalm\Internal\Codebase\ImpureFunctionsList::getCapabilities()` returns them.
- [BC] `Psalm\Context#removeMutableObjectVars()` takes the callee's capabilities as a second argument.

### `0.2.0`

Covers Psalm `7.0.0-beta20`

- [BC] Unused code detection is now based on a graph of references between code elements (`Psalm\Internal\Codebase\CodeUseGraph`), like taint analysis: a class, method, property or class constant is only considered used if it is reachable from an entry point (the public API, top-level code, free functions or code outside of the project). Code that is only referenced by other unused code, including cycles of otherwise unreferenced code, is now reported as unused.
- [BC] Plugins that want to mark code as used must reference it from a context that is itself alive (e.g. by passing a `Psalm\Context` with a `$calling_method_id` of a used method, or of a method outside of the project, to `Psalm\Codebase#methodExists`, `Psalm\Codebase#classExists` and similar methods): references made from unused project code no longer count.
- [BC] The reference-tracking methods of `Psalm\Internal\Provider\FileReferenceProvider` (`addMethodReferenceToClass`, `addNonMethodReferenceToClass`, `addMethodReferenceToClassMember`, `addFileReferenceToClassMember`, `isClassReferenced`, `isClassMethodReferenced`, `isClassPropertyReferenced`, `isMethodReturnReferenced`, `getClassLocations`, `getClassMethodLocations`, `getClassPropertyLocations` and their `getAll*`/`set*`/`add*` bulk variants) were removed: use `Psalm\Codebase#addReferenceToClass`, `Psalm\Codebase#addReferenceToFunctionLike`, `Psalm\Codebase#addReferenceToProperty`, `Psalm\Codebase#addReferenceToClassConstant` and `Psalm\Codebase#findReferencesTo*` instead.
- [BC] The `Psalm\Codebase#findReferencesToSymbol`, `findReferencesToMethod`, `findReferencesToProperty` and `findReferencesToClassLike` methods now return arrays keyed by location hash instead of lists.
- [BC] Method `Psalm\Internal\Codebase\Properties#propertyExists` now takes a `Psalm\Codebase` as its first parameter; use the new `Psalm\Codebase#propertyExists` wrapper instead.
- [BC] Method `Psalm\Internal\Codebase\ClassLikes#makeImmutable` was made private.
- [BC] `MissingPureAnnotation` is now reported after the whole codebase has been analysed, by `Psalm\Internal\Codebase\MutationLevelResolver`: the purity of a function-like calling unannotated functions or methods is inferred from the callees' inferred purity as a fixpoint over the call graph, so call chains in any order and recursive cycles (mutual recursion, recursive closures) are handled. Closures assigned to a variable are now reported too.
- [BC] `Psalm\StatementsSource#signalMutation()` and `Psalm\StatementsSource#signalMutationOnlyInferred()` (and their implementations in `Psalm\Internal\Analyzer\SourceAnalyzer`) take two additional parameters, `$callee_internal_mutations_ok` and `$callee_id`.
- [BC] `Psalm\Internal\FileManipulation\FunctionDocblockManipulator::getForFunction()` takes an additional optional `$docblock_anchor` parameter.
- [BC] Suppressing an unused-code issue (e.g. `@psalm-suppress UnusedClass` or `PossiblyUnusedMethod`) no longer makes the symbol an entry point: the suppression only silences the report for that symbol, and code that is only referenced from it is still reported as unused.
- [BC] All non-internal classes and interfaces (everything under `Psalm\` outside `Psalm\Internal`, except symbols marked `@internal`) are now marked `@api`. They are the supported public API and are treated as entry points by unused-code detection.
- [BC] `--find-unused-psalm-suppress` now also reports redundant `@psalm-suppress` annotations on classes, interfaces, traits and enums, and — when running with `--taint-analysis` — redundant `@psalm-suppress` of `Tainted*` issues. `Psalm\IssueBuffer::addUnusedSuppression()` gained a mandatory `$taint_analysis` parameter, and `Psalm\Internal\Analyzer\FunctionLikeAnalyzer::addSuppressedIssue()` (a redundant single-issue helper) was removed in favour of `addSuppressedIssues()`.


### `0.1.0`

Initial release, covers Psalm `7.0.0-beta17`.  

All breaking changes up to this point are listed in [vimeo/psalm#UPGRADING.md](https://github.com/vimeo/psalm/blob/master/UPGRADING.md).  

All breaking changes in future releases will be listed here.  

