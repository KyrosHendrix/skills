# Python patterns

Code examples for each rule in `SKILL.md`. The underlying principles are language-agnostic. See the [type-system-discipline](../../principles/type-system-discipline.md) and [boundary-discipline](../../principles/boundary-discipline.md) principles. Each "Do" block is self-contained and passes pyright in strict mode on Python 3.13.

## Discriminated unions

Model variants with a literal `kind`. Every variant has the field and each value is unique, so contradictory states can't exist.

```python
# Don't. A flag plus optionals lets loading=True and error="x" coexist.
@dataclass
class DiffState:
    loading: bool
    diff: str | None = None
    error: str | None = None
```

```python
from dataclasses import dataclass
from typing import Literal, assert_never


@dataclass(frozen=True)
class Loading:
    kind: Literal["loading"] = "loading"


@dataclass(frozen=True)
class Ready:
    diff: str
    kind: Literal["ready"] = "ready"


@dataclass(frozen=True)
class Failed:
    error: str
    kind: Literal["failed"] = "failed"


type DiffState = Loading | Ready | Failed


def render(state: DiffState) -> str:
    match state:
        case Loading():
            return "Loading..."
        case Ready(diff=diff):
            return diff
        case Failed(error=error):
            return f"Error: {error}"
        case _:
            assert_never(state)
```

Pick one discriminant name (`kind`) and stick to it. Class patterns narrow on their own; the `kind` field keeps the variant visible in serialized data.

## Branded types

`NewType` keeps two `str` values apart at type-check time. It is an identity function at runtime, so the parser is the only place that creates one.

```python
from typing import NewType
from uuid import UUID

AgentId = NewType("AgentId", str)


def parse_agent_id(raw: str) -> AgentId:
    try:
        return AgentId(str(UUID(raw)))
    except ValueError as err:
        raise ValueError(f"invalid agent id: {raw!r}") from err


def agent_url(agent_id: AgentId) -> str:
    return f"/agents/{agent_id}"
```

Never write `AgentId(raw)` outside the parser. `isinstance(x, AgentId)` raises `TypeError`.

## Constructive modeling

Build the type from parts that are all legal instead of checking a loose type at runtime.

```python
# Don't. Every caller must remember the empty case.
def pick_winner(entries: list[str]) -> str:
    if not entries:
        raise ValueError("no entries")
    return random.choice(entries)
```

```python
import random
from dataclasses import dataclass
from datetime import datetime, timedelta
from typing import NewType

type NonEmpty[T] = tuple[T, *tuple[T, ...]]
type Pairs[T] = tuple[tuple[T, T], ...]
Duration = NewType("Duration", timedelta)


def parse_duration(value: timedelta) -> Duration:
    if value < timedelta(0):
        raise ValueError(f"negative duration: {value}")
    return Duration(value)


def pick_winner(entries: NonEmpty[str]) -> str:
    return random.choice(entries)


@dataclass(frozen=True)
class Booking:
    start: datetime
    duration: Duration

    @property
    def end(self) -> datetime:
        return self.start + self.duration


winner = pick_winner(("ana", "ben"))
```

`NonEmpty` and `Pairs` make illegal lengths impossible to write. `timedelta` is signed, so the duration is branded non-negative once at the boundary; after that, no `start <= end` check is needed anywhere.

## Simplest total type

Keep the loose type while every operation on it is total. Strengthen only where it forces a lie.

```python
from collections.abc import Sequence
from decimal import Decimal

type NonEmpty[T] = tuple[T, *tuple[T, ...]]


def total(prices: Sequence[Decimal]) -> Decimal:
    return sum(prices, Decimal(0))


def first_price(prices: NonEmpty[Decimal]) -> Decimal:
    return prices[0]
```

`total` is fine on an empty list, so it keeps `Sequence`. `first_price` would need a `raise` on empty input, so it takes `NonEmpty`.

## `object` over `Any`

```python
# Don't. Any lets every typo through.
def handle(payload: Any) -> str:
    return payload["nmae"].upper()
```

```python
import json


def read_name(raw: str) -> str:
    payload: object = json.loads(raw)
    match payload:
        case {"name": str(name)}:
            return name
        case _:
            raise ValueError(f"payload has no string name: {raw!r}")
```

`json.loads` returns `Any`; annotate the result as `object` straight away so the checker forces a parse.

## Schemas before guards

Hand-written `isinstance` checks on a `dict` leave `dict[Unknown, Unknown]` in strict mode and drift from the type. A schema model owns both.

```python
from enum import StrEnum

from pydantic import BaseModel, ConfigDict


class Status(StrEnum):
    PENDING = "pending"
    PAID = "paid"


class Order(BaseModel):
    model_config = ConfigDict(frozen=True, strict=True, extra="forbid")

    id: str
    status: Status
    quantity: int


order = Order.model_validate_json('{"id": "A1", "status": "paid", "quantity": 2}')
```

`strict=True` stops pydantic coercing `true` to `1` or `"2"` to `2`. Under `strict=True`, `model_validate_json` still parses strings into `StrEnum`, `UUID`, and `datetime` fields, but `model_validate` on a Python dict expects real instances, so validate raw JSON text with `model_validate_json`. Use `extra="ignore"` instead of `"forbid"` on wire formats that gain fields over time. Use whichever library the repo already has. Don't add one for a single flat row; a `match` with mapping patterns is enough there.

## No `cast` or bare ignores

```python
# Don't. cast() checks nothing; a bad payload crashes far from here.
order = cast(Order, json.loads(raw))
```

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Order:
    order_id: str
    quantity: int


def parse_order(payload: object) -> Order:
    match payload:
        case {"id": str(order_id), "quantity": int(quantity)} if not isinstance(quantity, bool):
            return Order(order_id=order_id, quantity=quantity)
        case _:
            raise ValueError(f"not an order: {payload!r}")
```

`bool` is a subclass of `int`, so `int(quantity)` alone accepts `true`. When a third-party stub is wrong, use your checker's coded ignore with the reason: `# pyright: ignore[reportUnknownVariableType]` or `# type: ignore[no-untyped-call]`, followed by why.

## Narrowing hierarchy

Prefer the strongest narrowing the data allows: `match` > `isinstance` / `is None` > `TypeIs` > `cast`. Handle `None` once and pass the narrowed value on.

```python
import re

PATTERN = re.compile(r"v(\d+)")


def version(tag: str) -> int | None:
    found = PATTERN.fullmatch(tag)
    if found is None:
        return None
    return int(found.group(1))


def bump(tag: str) -> str:
    current = version(tag)
    if current is None:
        raise ValueError(f"not a version tag: {tag!r}")
    return f"v{current + 1}"
```

`version` treats no match as a normal result. `bump` can't continue without one, so it raises. Neither re-checks inside a helper it calls.

## Type guards

```python
# Don't. The name promises a check the body doesn't do.
def is_text_pair(pair: tuple[str, str] | tuple[int, int]) -> TypeIs[tuple[str, str]]:
    return True
```

```python
from typing import TypeIs


def is_text_pair(pair: tuple[str, str] | tuple[int, int]) -> TypeIs[tuple[str, str]]:
    return isinstance(pair[0], str)


def format_pair(pair: tuple[str, str] | tuple[int, int]) -> str:
    if is_text_pair(pair):
        return f"{pair[0]}={pair[1]}"
    return str(pair[0] + pair[1])
```

`TypeIs` narrows both branches: `tuple[str, str]` in the `if`, `tuple[int, int]` in the `else`. `TypeGuard` narrows only the `if`; use it only when the guarded type isn't a subtype of the input. Watch for `str`: it is a `Sequence[str]`, so a guard for "list of strings" must reject a bare `str`.

## Exhaustiveness

```python
from enum import StrEnum
from typing import assert_never


class Status(StrEnum):
    PENDING = "pending"
    PAID = "paid"
    SHIPPED = "shipped"


def label(status: Status) -> str:
    match status:
        case Status.PENDING:
            return "Awaiting payment"
        case Status.PAID:
            return "Ready to ship"
        case Status.SHIPPED:
            return "On its way"
        case _:
            assert_never(status)
```

Add a member to `Status` and the checker flags `assert_never`. Without the default arm, a `match` used as a statement, or in a function returning `None`, falls through silently.

## Boundary validation

Parse once where data crosses in. Everything past the parse takes the domain type.

```python
import argparse
from collections.abc import Mapping, Sequence
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class Settings:
    input_dir: Path
    limit: int


def parse_args(argv: Sequence[str]) -> Settings:
    parser = argparse.ArgumentParser()
    parser.add_argument("input_dir", type=Path)
    parser.add_argument("--limit", type=int, default=10)
    raw: Mapping[str, object] = vars(parser.parse_args(argv))
    match raw:
        case {"input_dir": Path() as input_dir, "limit": int(limit)} if not isinstance(limit, bool):
            pass
        case _:
            parser.error(f"unexpected arguments: {raw!r}")
    if not input_dir.is_dir():
        parser.error(f"{input_dir} is not a directory")
    if limit < 1:
        parser.error("--limit must be at least 1")
    return Settings(input_dir=input_dir, limit=limit)


def run(settings: Settings) -> list[Path]:
    return sorted(settings.input_dir.iterdir())[: settings.limit]
```

`argparse.Namespace` attributes are `Any`, so a typo like `namespace.inptu_dir` type-checks. Reading it as `Mapping[str, object]` forces a real narrow. `run` doesn't re-check the directory or the limit; `Settings` already guarantees both.

## Schema-derived types

When a schema or model owns a shape, use it. Python has no `Pick` or `Omit`, so don't hand-copy a twin class.

```python
# Don't. A hand-copied twin of the API model drifts when the schema changes.
@dataclass
class OrderSummary:
    id: str
    status: str
```

```python
import logging
from collections.abc import Callable, Sequence
from functools import wraps
from typing import Literal, Protocol, TypedDict, Unpack

from pydantic import BaseModel

logger = logging.getLogger(__name__)


class Order(BaseModel):
    id: str
    status: Literal["pending", "paid"]
    total_cents: int


def summarize(order: Order) -> str:
    return f"{order.id}: {order.status}"


class HasId(Protocol):
    @property
    def id(self) -> str: ...


def audit_key(item: HasId) -> str:
    return f"audit:{item.id}"


def logged[**P, R](fn: Callable[P, R]) -> Callable[P, R]:
    name = getattr(fn, "__name__", repr(fn))

    @wraps(fn)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        logger.debug("calling %s", name)
        return fn(*args, **kwargs)

    return wrapper


class RetryOptions(TypedDict, total=False):
    attempts: int
    backoff: float


def fetch(url: str, **options: Unpack[RetryOptions]) -> str:
    return f"{url} {options}"


def fetch_all(urls: Sequence[str], **options: Unpack[RetryOptions]) -> list[str]:
    return [fetch(url, **options) for url in urls]


key = audit_key(Order(id="A1", status="paid", total_cents=100))
```

Pass the owning model (here a pydantic model; in practice often generated from OpenAPI, protobuf, or an ORM) instead of a parallel class. Use a `Protocol` only to let a helper accept any object with a few attributes, such as `audit_key`. `logged` keeps the wrapped function's exact signature through `ParamSpec`. `fetch_all` shares `fetch`'s options through `Unpack` instead of listing them twice.

## Keyword-only args

```python
# Don't. transfer(b, a, amount) compiles and moves money the wrong way.
def transfer(source: AccountId, target: AccountId, amount: Decimal) -> None: ...
```

```python
import logging
from decimal import Decimal
from typing import NewType

logger = logging.getLogger(__name__)
AccountId = NewType("AccountId", str)


def parse_account_id(raw: str) -> AccountId:
    cleaned = raw.strip()
    if not cleaned:
        raise ValueError("empty account id")
    return AccountId(cleaned)


def transfer(*, source: AccountId, target: AccountId, amount: Decimal) -> None:
    logger.info("transfer %s from %s to %s", amount, source, target)


transfer(source=parse_account_id("acct-1"), target=parse_account_id("acct-2"), amount=Decimal("5"))
```

## Closed sets

A `StrEnum` lists its values once and parses from text. A `Literal` needs a second tuple to parse, and the two drift.

```python
from enum import StrEnum


class Mode(StrEnum):
    FAST = "fast"
    SAFE = "safe"


def parse_mode(raw: str) -> Mode:
    try:
        return Mode(raw.strip().casefold())
    except ValueError as err:
        allowed = ", ".join(Mode)
        raise ValueError(f"unknown mode {raw!r}, expected one of {allowed}") from err
```

## `Path` for files

```python
import os
from pathlib import Path


def load_config(path: str | os.PathLike[str]) -> str:
    return read_text(Path(path))


def read_text(path: Path) -> str:
    return path.read_text(encoding="utf-8")
```

The public entry point accepts what callers have and converts it once. Internal helpers take `Path`.

## Modern annotations

```python
# Don't.
from typing import Dict, List, Optional, TypeVar
T = TypeVar("T")
def first(items: List[T]) -> Optional[T]: ...
```

```python
from collections.abc import Sequence

type Pair[T] = tuple[T, T]


def first[T](items: Sequence[T]) -> T | None:
    return items[0] if items else None


def swap[T](pair: Pair[T]) -> Pair[T]:
    return (pair[1], pair[0])
```

## Real tests

```python
# Don't. The mock passes even if the real parser is broken.
def test_load(mocker: MockerFixture) -> None:
    mocker.patch("app.load_rows", return_value=[{"id": "1"}])
```

```python
from pathlib import Path


def count_rows(path: Path) -> int:
    return sum(1 for line in path.read_text(encoding="utf-8").splitlines() if line.strip())


def test_count_rows_skips_blank_lines(tmp_path: Path) -> None:
    data = tmp_path / "rows.csv"
    data.write_text("a\n\nb\n", encoding="utf-8")
    assert count_rows(data) == 2
```

Fake only what can't run locally: network, paid APIs, the clock, hardware.

## Structured logging

```python
import logging
from pathlib import Path

logger = logging.getLogger(__name__)


def import_rows(path: Path) -> int:
    imported = 0
    for number, line in enumerate(path.read_text(encoding="utf-8").splitlines(), start=1):
        if "," not in line:
            logger.warning("skipped row %d of %s: no delimiter", number, path.name)
            continue
        imported += 1
    logger.info("imported %d rows from %s", imported, path.name)
    return imported
```

Use `%`-style arguments, so the message is formatted only when emitted. Log ids and file names, not tokens, passwords, or personal data. Configure handlers once in the application entry point.
