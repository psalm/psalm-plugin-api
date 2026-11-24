# psalm-plugin-api

This package must be required by [Psalm](https://psalm.dev) plugins in order to require a specific version of the Psalm plugin API.  

This is a separate, empty metapackage: its major version will be bumped every time a breaking change occurs within Psalm's plugin API, breaking changes will be listed in this README; minors will also be bumped when adding new features to Psalm's plugin API.  

This allows breaking changes within Psalm's plugin API, without requiring a major in Psalm itself (which will be reserved only for breaking changes in the CLI API or other major behavioral changes).  

## Changelog

### `0.5.2`

- [BC] `Psalm\Type\TaintKind::INPUT_SQL` now only covers SQL that breaks out of quotes (vimeo/psalm#14400). The new `INPUT_SQL_UNQUOTED` covers SQL syntax outside of quotes, which `real_escape_string()` and the other escapes that rely on quotes don't remove. In annotations, `sql` names both, and `sql_in_quotes` names `INPUT_SQL` alone. A plugin declaring an SQL sink, or an escape that adds the quotes itself, with the constants should use `INPUT_SQL | INPUT_SQL_UNQUOTED`; an escape that only works inside quotes keeps `INPUT_SQL`. This change shipped in Psalm 7.13.2 with plugin API `0.5.1`, so it is listed here instead of in a minor release.
- New taint kinds `html_attribute` (`TaintKind::INPUT_HTML_ATTRIBUTE`, reported as `TaintedHtmlAttribute`) and `url_scheme` (`TaintKind::INPUT_URL_SCHEME`, reported as `TaintedUrlScheme`), part of `input` (vimeo/psalm#14400). `htmlspecialchars()` and `htmlentities()` keep them, and Psalm reports them where the markup before a value puts it in an unquoted attribute value, or at the start of a URL a browser runs as script. An escaping function that leaves no whitespace, or that fixes the scheme, can declare `@psalm-taint-escape html_attribute` or `@psalm-taint-escape url_scheme`. `TaintKind::USER_SECRET` and `SYSTEM_SECRET` move up three bits, and `ALL_INPUT`, `BUILTIN_TAINT_COUNT` and `TAINT_NAMES` change accordingly: use them symbolically.
- `@template-contravariant` (vimeo/psalm#14659): `Psalm\Storage\ClassLikeStorage#$template_contravariants` holds the offsets of the class templates declared with it (`array<int, true>|null`), and `ClassLikeStorage#getTemplateVariance(int $offset)` returns their variance as a `Psalm\Storage\TemplateVariance` case (`Invariant`, `Covariant`, `Contravariant`). `#$template_covariants` keeps its meaning, but `false` there no longer means invariant, since a contravariant template is `false` too: read `getTemplateVariance()`. A plugin that declares a contravariant template sets its offset in `#$template_contravariants`, which wins over `#$template_covariants`.

### `0.5.1`

- `Psalm\Issue\CodeIssue::CWE_IDS` lists the ids of the [CWE](https://cwe.mitre.org/) weaknesses an issue flags, the most specific first (vimeo/psalm#14156). A plugin issue declares its own: `public const CWE_IDS = [89];`. Reports show them after the message, the JSON report in `cwe_ids`, and the SARIF report as `external/cwe/cwe-NNN` rule tags; the SARIF `ruleId` is now the issue type rather than its shortcode.
- New taint kind `redirect` (`Psalm\Type\TaintKind::INPUT_REDIRECT`, reported as `TaintedRedirect`), part of `input` (vimeo/psalm#14158). Use `@psalm-taint-sink redirect $url` for the redirects of a framework, along with `url_component` and `url_path` to report what a value injects after a fixed server. `TaintKind::USER_SECRET` and `SYSTEM_SECRET` move up one bit, and `ALL_INPUT`, `BUILTIN_TAINT_COUNT` and `TAINT_NAMES` change accordingly: use them symbolically. What is stored in `$_SESSION` flows into what reads it back; frameworks' session services can do the same with a [keyed store](https://psalm.dev/docs/security_analysis/taint_flow/#keyed-stores) (`@psalm-flow ($value) -> Session::$data[$key]`).
- New security issues a plugin can report for the APIs of a framework (vimeo/psalm#14160): `Psalm\Issue\WeakHash`, `WeakCipher`, `WeakRandom`, `InsecureCookie` (a cookie set without the secure flag) and `XmlExternalEntity`.
- `Psalm\Storage\Assertion\MatchesPattern` says that a value matches a regular expression telling the characters it holds: it doesn't narrow the type, but the value can't carry its `$removed_taints` (vimeo/psalm#14161).

### `0.5.0`

- [BC] `Psalm\Storage\FunctionLikeStorage#$allow_named_arg_calls` was replaced by `#$named_arguments`, a `Psalm\Storage\NamedArguments` case or null (vimeo/psalm#12389). It comes from the docblock of the function, of its class or of the method it overrides: `NOT_ALLOWED` for `@no-named-arguments` (and for internal code when `allowInternalNamedArgumentsCalls="false"`), `REQUIRED` for `@arguments-by-name`, `REQUIRED_FOR_OPTIONAL` for `@optional-arguments-by-name` and `STABLE` for `@stable-parameter-names`. Call `FunctionLikeStorage#allowsNamedArguments()` instead of reading the property, and set `#$named_arguments` to `NamedArguments::NOT_ALLOWED` instead of setting it to `false`.
- [BC] `Psalm\Plugin\DynamicFunctionStorage#$allow_named_arg_calls` was replaced by `#$named_arguments` in the same way: set it to `NamedArguments::NOT_ALLOWED` instead of setting `#$allow_named_arg_calls` to `false`.
- `Psalm\Storage\ClassLikeStorage#$named_arguments` holds the tag of a class, interface, trait or enum, which its methods without a tag of their own use (the private methods of a class only take `@no-named-arguments` from it). `Psalm\Storage\FunctionLikeStorage#requiresNamedArgument()` tells whether the argument of a parameter must be passed by name; calls that pass it by position are reported as `Psalm\Issue\PositionalArgumentNotAllowed`.

### `0.4.6`

- `Psalm\Config#$mark_all_non_internal_code_as_api` is set by `markAllNonInternalCodeAsApi="true"`: unused-code detection then treats every project class that isn't internal (empty `Psalm\Storage\ClassLikeStorage#$internal`) as `@api`, apart from the code the library doesn't ship. A plugin can keep the code users of a library can't use, such as its test suites, out of that API by setting `ClassLikeStorage#$internal` in `afterCodebasePopulated()`.

### `0.4.5`

- `Psalm\Codebase#addReportableFiles()` lets Psalm report issues in files it neither scans nor analyzes, such as the templates a plugin compiles to PHP and reports the issues of the compiled file on. Use it instead of writing to `ProjectAnalyzer#$project_files`, which also queues the files for analysis. Call it in the plugin entry point (`$registration->codebase`): an issue is checked in the process that raises it, and files added there are reportable in every scanning and analysis process.

### `0.4.4`

- `Psalm\Storage\ClassLikeStorage#$namedMixins` and `#$templatedMixins` hold only the mixins the class declares (and those plugins add to them); a class no longer gets the lists of its parent. The mixins of its traits and parent classes are looked up at analysis time: those of the class first, then those of its traits, then those of its parent class, the first declared winning. Mixins a plugin appends come after the declared ones.
- `ClassLikeStorage#$transitiveNamedMixins`, `#$inheritedNamedMixinDeclaringClasses` (both always empty), `#$mixin_declaring_fqcln` (never set or read), `#getNamedMixinsForLookup()` (now returns `#$namedMixins`), `#getNamedMixinDeclaringClass()` (now the class whose templates the mixin's type arguments use, else the class itself) and `#getOwnMixins()` (now the mixins the class declares, then those plugins added) are deprecated and will be removed in a later release: the mixins of a mixin are looked up along with it, and each mixin is bound in the context of the class, trait or parent class declaring it. Read `#$namedMixins` and `#$templatedMixins`.

### `0.4.3`

- `Psalm\Plugin\EventHandler\Event\BeforeFileAnalysisEvent#markFileAsCliOnly()` says that a file's top-level code and the closures in it only run from the command line, like code marked `@psalm-cli-only` (what it prints isn't reported as `TaintedHtml`, and it may call `@psalm-cli-only` code); `#isFileCliOnly()` tells whether it does (a plugin marked it, or code only running from the command line includes it).

### `0.4.2`

- `Psalm\Plugin\PluginFixableIssuesInterface` lets a plugin entry point declare the issues it can fix (`getFixableIssues()`, a list of issue classes): Psalter then accepts them in `--issues`, fixes them with `--issues=all` and lists them with `--list-supported-issues`.
- `Psalm\Codebase#shouldFixIssue()` tells a plugin whether to add its file replacements for an issue: when Psalter was asked to fix its type and neither the given suppressed issues nor `psalm.xml` suppress it.

### `0.4.1`

- `Psalm\Codebase#getReceiverTaintPath()` gives the path along which the taints of the object a method of a builtin class (`ArrayObject`, `ArrayIterator`, `SplObjectStorage`, `Generator`, …, or a class extending one) is called on flow into what the method returns (`=`, `arrayvalue-fetch`, `arrayvalue-fetch-'key'` for a literal key, `arraykey-fetch`, `return`), or null if the method returns none of them.

### `0.4.0`

Covers the Psalm 7 master branch after `7.0.0-rc2`.

- [BC] A new capability, `time` (`Psalm\Storage\Capabilities::TIME`, 128), covers reading the clock (`time()`, `microtime()`, `date()` without a timestamp, …), which used to cost `io`. `Capabilities::ALL` (`impure`) is now 255, and so are the defaults of every capability set: `Psalm\Context#$capabilities`, `Psalm\Storage\FunctionLikeStorage#$capabilities`, `Psalm\Storage\ClassLikeStorage#$capabilities`, `Psalm\Storage\MethodStorage#$containing_class_capabilities`, `Psalm\Plugin\DynamicFunctionStorage#$capabilities`, the `$purity` of the `TCallable` and `TClosure` constructors and `Context#removeMutableObjectVars()`'s `$callee_capabilities`. `Capabilities::TAINT_SPECIALIZED` changed accordingly.
- [BC] `Psalm\Storage\Capabilities::allows()` was renamed to `allowsAnyOf()`. `Capabilities::equals()` compares two capability sets exactly, and capability sets are typed as `CapabilitySet` int-masks.
- [BC] `Psalm\Type\Union#getTaintsToRemove()`, `getAtomicTaintsToRemove()` and `getArrayKeyTaintsToRemove()` take the taints of user input, `Psalm\Codebase#getInputTaints()`, which include the custom taints `psalm.xml` adds to `input`.
- [BC] The `$taints` of `Psalm\Codebase#addTaintSource()` and `addTaintSink()` default to `null`, the taints of user input (`Codebase#getInputTaints()`), rather than the built-in ones.
- `Psalm\Plugin\EventHandler\MethodCapabilitiesProviderInterface` lets a plugin give the capabilities of a method call.
- `@psalm-taint-sink <kind> $param[*]` makes the values of an array parameter a sink, but not its keys.

### `0.3.0`

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

