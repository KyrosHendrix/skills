---
name: python-best-practices
description: Python best practices. Use when reading or editing any .py or .pyi file.
---

# Python best practices

Apply the [type-system-discipline](../principles/type-system-discipline.md) principle first. Assumes Python 3.13+; when the repo's `requires-python` is older, use only the features it allows (version gates are noted below).

| Rule | Summary |
|------|---------|
| Discriminated unions | Model variants as `@dataclass(frozen=True)` classes, each with a `kind: Literal["..."]` field, joined in a union. Narrow with `match` (3.10+). No optional-field bags. |
| Branded types | Give distinct `str` or `int` domain values their own `NewType` so they can't be swapped. `NewType` does no runtime check and fails `isinstance`, so create it only inside a validating parser at the boundary. |
| Constructive modeling | Build the shape so the illegal value can't be constructed. `tuple[T, *tuple[T, ...]]` (3.11+) for non-empty, `tuple[T, T]` for a pair, `start` plus a duration branded non-negative for a range. Not a runtime guard. |
| Simplest total type | Keep `Sequence[T]` while every operation on it stays total. Strengthen to a non-empty type only where the loose type forces a `cast`, an `assert`, a `# type: ignore`, or a "should never happen" `raise`. |
| `object` over `Any` | External data is `object` until parsed. `Any` switches the checker off for everything it touches. `bool` is an `int`, so an `int` check on parsed data must also exclude `bool`. |
| Schemas before guards | Before hand-writing `isinstance` checks on nested dicts, use the repo's schema library (pydantic, msgspec) and let one model own both validation and the type. `TypedDict` validates nothing at runtime. |
| No `cast` or bare ignores | `typing.cast`, `# type: ignore`, and `assert` used to silence the checker (stripped under `python -O`) are crashes waiting. Narrow or parse instead. When a library stub is wrong, use your checker's coded ignore (`# pyright: ignore[rule]` or `# type: ignore[code]`) with the reason. Python has no `satisfies`; annotate the variable with the target type instead of casting, accepting that literals widen to the annotation. |
| Narrowing hierarchy | `match` on a discriminant or class pattern > `isinstance` / `is None` > `TypeIs` guard > `cast` after validation. Handle `X \| None` once, by returning, defaulting, or raising, then pass the narrowed value on. |
| Type guards | Use `TypeIs[T]` (3.13+) over `TypeGuard[T]` (3.10+); it narrows both branches. A guard must verify its claim. A lying guard is worse than `cast` because the bug hides behind a name that says it's safe. Name them `is_x` or `has_x`. |
| Exhaustiveness | End a `match` over a union, `Literal`, or enum with `case _: assert_never(x)` (3.11+) so the checker errors when a new variant is added. |
| Boundary validation | Parse where data crosses in (JSON, config, CLI args, env, API rows) into a named domain type. `Mapping[str, object]` stops at that parse; `dict[str, object]` is invariant and rejects `dict[str, str]` callers. Trust types inside, with no re-validation. See the [boundary-discipline](../principles/boundary-discipline.md) principle. |
| Schema-derived types | Don't redeclare a shape a schema or model already owns (pydantic, OpenAPI, protobuf, ORM); pass that model. Python has no `Pick` or `Omit`: use a `Protocol` only to accept a minimal set of attributes, `ParamSpec` (3.10+; `[**P]` syntax 3.12+) to forward a signature, and `Unpack[SomeTypedDict]` (3.12+) for shared keyword arguments. |
| Keyword-only args | Put `*` before parameters that share a type or aren't obvious from position, so callers pass names. Skip on hot paths and one-argument helpers. |
| Closed sets | A status, mode, or kind is a `StrEnum` (3.11+) or a `Literal`, never a bare `str`. Prefer `StrEnum` when you must parse from text, so the values are listed once. |
| `Path` for files | Convert `str` or `os.PathLike[str]` to `pathlib.Path` at the boundary. Internal functions take `Path`, not `str \| Path`. |
| Modern annotations | `list[str]`, `X \| None` (3.10+), `type Alias = ...` and `def f[T]` (3.12+). No `List`, `Dict`, `Optional`, `Union`, or `TypeVar` boilerplate. Annotate every parameter except `self` and `cls`, and every public return. |
| Real tests | Don't mock what you can run. Use the repo's test runner with real files in `tmp_path`. Fake only what can't run locally: network, paid APIs, clocks, hardware. |
| Structured logging | `logger = logging.getLogger(__name__)` per module, or the repo's structured logger, with enough non-secret context and ids to debug. No `print` and no `logging.basicConfig` in library code; `print` is for a CLI's stdout only. |

Examples: `references/patterns.md`.
