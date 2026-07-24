# Part 5: Functions as First-Class Objects, Closures, `*args`, and `**kwargs`

Python functions are more than reusable blocks of instructions. They are also values.

That means a function can be:

- Stored in a variable
- Passed into another function
- Returned from another function
- Kept in a list or dictionary
- Registered as a callback for an event

This is called being a **first-class object**.

In this part, we will use first-class functions to add a small event system to our library package. When a catalog item is added, checked out, or returned, the `Library` will emit an event. Other code can choose to listen to those events.

This pattern appears throughout real applications:

- A web framework calls your function when an HTTP request arrives.
- A test runner calls your test functions.
- A message queue calls your message handler.
- An API client calls hooks before or after an HTTP request.
- A UI calls a function when a user clicks a button.

---

## What You Will Build

By the end of this part, the project will include:

```text
pythonic-craftsmanship/
├── src/
│   └── library/
│       ├── __init__.py
│       ├── book.py
│       ├── catalog.py
│       ├── digital_book.py
│       ├── events.py
│       ├── lending_item.py
│       ├── reports.py
│       ├── search.py
│       └── snapshots.py
└── examples/
    ├── part_5_callbacks_demo.py
    ├── part_5_closures_demo.py
    └── part_5_variadic_arguments_demo.py
```

The new functionality will include:

- A typed `LibraryEvent` data object.
- Callback functions that receive library events.
- A callback registry that supports multiple listeners.
- Closures that remember configuration from the surrounding scope.
- Flexible functions using `*args` and `**kwargs`.
- Careful handling of callback failures so one optional listener does not break the core catalog operation.

---

# Step 1: Treat Functions as Values

## The Target

We are creating:

```text
examples/part_5_callbacks_preview.py
```

This small program demonstrates the basic first-class-function behavior before we add it to the library package.

## The Concept

A function name refers to a function object.

For example:

```python
def announce(message: str) -> None:
    print(message)
```

The name `announce` can be passed to another function without parentheses:

```python
handler = announce
```

The parentheses matter:

```python
handler = announce
```

means:

> Store the function itself.

But:

```python
handler = announce("Hello")
```

means:

> Call the function now, then store its return value.

This distinction is essential for callbacks.

A **callback** is a function that you give to another part of the program so it can call your function later.

Think of leaving your phone number with a delivery company:

- You provide a number now.
- The company contacts you later when the package arrives.

The callback is the phone number. The event is the package arrival.

## The Implementation

### `examples/part_5_callbacks_preview.py`

```python
"""Demonstrate that Python functions can be stored and passed as values."""

from collections.abc import Callable


def print_notification(message: str) -> None:
    """Print a notification message."""
    print(f"NOTIFICATION: {message}")


def write_notification_to_memory(
    destination: list[str],
) -> Callable[[str], None]:
    """Return a callback that stores future messages in destination.

    The returned function is a closure. It remembers the destination list even
    after write_notification_to_memory() has returned.
    """

    def store_message(message: str) -> None:
        destination.append(message)

    return store_message


def deliver_message(message: str, handler: Callable[[str], None]) -> None:
    """Deliver one message by calling the supplied callback function."""
    handler(message)


def main() -> None:
    """Store functions, pass them around, and call them later."""
    notification_handler = print_notification

    deliver_message("A new item was added.", notification_handler)

    saved_messages: list[str] = []
    memory_handler = write_notification_to_memory(saved_messages)

    deliver_message("A book was checked out.", memory_handler)
    deliver_message("A book was returned.", memory_handler)

    print("\nMessages stored by the callback:")
    for saved_message in saved_messages:
        print(f"- {saved_message}")


if __name__ == "__main__":
    main()
```

## The Verification

Run:

```bash
python examples/part_5_callbacks_preview.py
```

Expected output:

```text
NOTIFICATION: A new item was added.

Messages stored by the callback:
- A book was checked out.
- A book was returned.
```

The `memory_handler` function was created once, stored in a variable, and called twice later. It retained access to the `saved_messages` list.

That remembered surrounding state is the foundation of a closure.

---

# Step 2: Define Typed Library Events

## The Target

We are creating:

```text
src/library/events.py
```

This module will define:

- `LibraryEvent`: information about something that happened.
- `EventHandler`: the type of a callback that can receive an event.
- `EventDispatchFailure`: information about a callback that failed.

## The Concept

Passing loose dictionaries between components is possible:

```python
{
    "event_type": "item_checked_out",
    "identifier": "BOOK-001",
}
```

But dictionaries are easy to misspell and difficult for type checkers to understand.

For structured event data, a **data class** is a better fit.

A data class is a class primarily used to store data. The `@dataclass` decorator generates useful methods automatically, including an initializer and readable representation.

For example:

```python
@dataclass(frozen=True)
class LibraryEvent:
    event_type: str
    item_identifier: str
```

The `frozen=True` setting makes the object immutable after creation. Event records should represent what happened at a point in time; listeners should not be able to rewrite history by changing them.

## The Implementation

### `src/library/events.py`

```python
"""Typed event definitions for library catalog activity."""

from __future__ import annotations

from collections.abc import Callable
from dataclasses import dataclass
from datetime import datetime
from typing import Literal

LibraryEventType = Literal[
    "item_added",
    "item_checked_out",
    "item_returned",
]

EventHandler = Callable[["LibraryEvent"], None]


@dataclass(frozen=True, slots=True)
class LibraryEvent:
    """Describe one completed catalog action.

    Attributes:
        event_type: The kind of action that completed.
        item_identifier: The stable identifier of the affected item.
        item_title: The human-readable title of the affected item.
        occurred_at: The UTC timestamp at which the event was created.
    """

    event_type: LibraryEventType
    item_identifier: str
    item_title: str
    occurred_at: datetime


@dataclass(frozen=True, slots=True)
class EventDispatchFailure:
    """Describe a callback failure without interrupting catalog behavior.

    Attributes:
        handler_name: A readable name for the failing callback.
        event: The event being delivered when the callback failed.
        error: The exception raised by the callback.
    """

    handler_name: str
    event: LibraryEvent
    error: Exception
```

## The Verification

Run:

```bash
python -c "from datetime import UTC, datetime; from library.events import LibraryEvent; event = LibraryEvent(event_type='item_added', item_identifier='BOOK-001', item_title='Clean Code', occurred_at=datetime.now(UTC)); print(event)"
```

Expected output will include the current timestamp, similar to:

```text
LibraryEvent(event_type='item_added', item_identifier='BOOK-001', item_title='Clean Code', occurred_at=datetime.datetime(..., tzinfo=datetime.timezone.utc))
```

Now verify that events are immutable:

```bash
python -c "from datetime import UTC, datetime; from library.events import LibraryEvent; event = LibraryEvent(event_type='item_added', item_identifier='BOOK-001', item_title='Clean Code', occurred_at=datetime.now(UTC)); event.item_title = 'Changed Title'"
```

Expected result:

```text
dataclasses.FrozenInstanceError: cannot assign to field 'item_title'
```

---

# Step 3: Add Callback Registration and Event Dispatching to `Library`

## The Target

We are extending:

```text
src/library/catalog.py
```

The `Library` class will gain these public methods:

- `subscribe(handler)`
- `unsubscribe(handler)`
- `event_dispatch_failures()`

It will emit events after successful actions:

- Adding an item
- Checking out an item
- Returning an item

## The Concept

The catalog’s main responsibility is still managing items. It should not need to know whether another part of the application wants to:

- Print activity to a terminal
- Store analytics data
- Send an email
- Write an audit log
- Update a dashboard

Instead, the catalog announces what happened through events.

This is a lightweight version of the **observer pattern**:

```text
Library completes an action
        ↓
Library emits an event
        ↓
Subscribed callback functions receive it
```

The library should protect its primary work from optional callback problems.

For example, a checkout must remain successful even if an analytics callback has a bug. We will capture callback errors in `EventDispatchFailure` records rather than rolling back an already completed checkout.

This policy is appropriate for **non-critical observers**. If a listener were essential—for example, a payment system recording a financial transaction—you would design a stronger delivery and retry strategy.

## The Implementation

Replace the complete contents of `src/library/catalog.py`.

### `src/library/catalog.py`

```python
"""Catalog services for managing a collection of lending items."""

from __future__ import annotations

from collections import deque
from collections.abc import Callable, Iterator
from datetime import UTC, datetime
from itertools import islice

from library.events import EventDispatchFailure, EventHandler, LibraryEvent, LibraryEventType
from library.lending_item import LendingItem
from library.search import (
    CatalogPageIterator,
    iter_available_identifiers,
    iter_available_items,
    iter_items_excluding,
    iter_items_matching_title,
)


class Library:
    """Manage a collection of lending items indexed by identifier."""

    def __init__(self, name: str, activity_history_limit: int = 100) -> None:
        """Create an empty library catalog.

        Args:
            name: The public name of the library.
            activity_history_limit: Maximum number of recent actions and
                callback failures to keep.

        Raises:
            ValueError: If name is blank or the history limit is less than 1.
        """
        cleaned_name = name.strip()

        if not cleaned_name:
            raise ValueError("Library name cannot be blank.")

        if activity_history_limit < 1:
            raise ValueError("Activity history limit must be at least 1.")

        self.name = cleaned_name
        self._items_by_identifier: dict[str, LendingItem] = {}
        self._recent_activity: deque[str] = deque(maxlen=activity_history_limit)

        # Callbacks are optional observers. A list preserves subscription order.
        self._event_handlers: list[EventHandler] = []
        self._event_dispatch_failures: deque[EventDispatchFailure] = deque(
            maxlen=activity_history_limit
        )

    def __repr__(self) -> str:
        """Return a detailed representation intended for developers."""
        return f"Library(name={self.name!r}, item_count={len(self)!r})"

    def __str__(self) -> str:
        """Return a concise representation intended for library users."""
        return self.name

    def __len__(self) -> int:
        """Return the number of items stored in the catalog."""
        return len(self._items_by_identifier)

    def __iter__(self) -> Iterator[LendingItem]:
        """Iterate over catalog items in the order they were added."""
        return iter(self._items_by_identifier.values())

    def __contains__(self, identifier: object) -> bool:
        """Return whether an identifier exists in this catalog."""
        return isinstance(identifier, str) and identifier in self._items_by_identifier

    def _record_activity(self, message: str) -> None:
        """Record one timestamped event in the bounded activity history."""
        timestamp = datetime.now(UTC).isoformat(timespec="seconds")
        self._recent_activity.append(f"{timestamp} | {message}")

    def _emit_event(self, event_type: LibraryEventType, item: LendingItem) -> None:
        """Notify subscribed handlers after a completed catalog action.

        Callback failures are recorded rather than raised. The catalog action
        already completed successfully and should not be undone because an
        optional observer failed.
        """
        event = LibraryEvent(
            event_type=event_type,
            item_identifier=item.identifier,
            item_title=item.title,
            occurred_at=datetime.now(UTC),
        )

        # tuple(...) creates a stable snapshot. A handler can safely unsubscribe
        # itself while receiving an event without changing this dispatch loop.
        for handler in tuple(self._event_handlers):
            try:
                handler(event)
            except Exception as error:
                handler_name = getattr(handler, "__name__", type(handler).__name__)
                self._event_dispatch_failures.append(
                    EventDispatchFailure(
                        handler_name=handler_name,
                        event=event,
                        error=error,
                    )
                )
                self._record_activity(
                    f'Event handler "{handler_name}" failed while processing '
                    f'"{event.event_type}".'
                )

    def subscribe(self, handler: EventHandler) -> None:
        """Register an event handler if it is not already registered.

        Raises:
            TypeError: If handler is not callable.
        """
        if not callable(handler):
            raise TypeError("Event handler must be callable.")

        if handler not in self._event_handlers:
            self._event_handlers.append(handler)

    def unsubscribe(self, handler: EventHandler) -> bool:
        """Remove a registered handler.

        Returns:
            True if the handler was registered and removed; otherwise False.
        """
        try:
            self._event_handlers.remove(handler)
        except ValueError:
            return False

        return True

    def event_dispatch_failures(self) -> tuple[EventDispatchFailure, ...]:
        """Return an immutable snapshot of optional callback failures."""
        return tuple(self._event_dispatch_failures)

    def add_item(self, item: LendingItem) -> None:
        """Add a lending item to the catalog.

        Raises:
            TypeError: If item is not a LendingItem instance.
            ValueError: If another item already uses the same identifier.
        """
        if not isinstance(item, LendingItem):
            raise TypeError("Library items must inherit from LendingItem.")

        if item.identifier in self._items_by_identifier:
            raise ValueError(
                f'An item with identifier "{item.identifier}" already exists '
                f"in {self.name}."
            )

        self._items_by_identifier[item.identifier] = item
        self._record_activity(f'Added item "{item.identifier}".')
        self._emit_event("item_added", item)

    def find_item(self, identifier: str) -> LendingItem:
        """Return the catalog item with the given identifier.

        Raises:
            KeyError: If the item does not exist.
        """
        cleaned_identifier = identifier.strip()

        try:
            return self._items_by_identifier[cleaned_identifier]
        except KeyError as error:
            raise KeyError(
                f'No item with identifier "{cleaned_identifier}" exists in {self.name}.'
            ) from error

    def available_items(self) -> list[LendingItem]:
        """Return all catalog items that can currently be checked out."""
        return list(self.iter_available_items())

    def iter_available_items(self) -> Iterator[LendingItem]:
        """Yield each item that can currently be checked out."""
        return iter_available_items(self)

    def iter_available_identifiers(self) -> Iterator[str]:
        """Yield identifiers for items that can currently be checked out."""
        return iter_available_identifiers(self)

    def search_title(self, query: str) -> Iterator[LendingItem]:
        """Yield items with titles containing query, case-insensitively."""
        return iter_items_matching_title(self, query)

    def first_items(self, limit: int) -> Iterator[LendingItem]:
        """Yield no more than limit catalog items.

        Raises:
            ValueError: If limit is negative.
        """
        if limit < 0:
            raise ValueError("Limit cannot be negative.")

        return islice(self, limit)

    def iter_items_excluding(
        self,
        predicate: Callable[[LendingItem], bool],
    ) -> Iterator[LendingItem]:
        """Yield catalog items that do not match predicate."""
        return iter_items_excluding(self, predicate)

    def iter_pages(self, page_size: int) -> CatalogPageIterator:
        """Return an iterator that yields catalog items in fixed-size pages."""
        return CatalogPageIterator(items=tuple(self), page_size=page_size)

    def check_out_item(self, identifier: str) -> LendingItem:
        """Check out an item and return its updated object."""
        item = self.find_item(identifier)
        item.check_out()
        self._record_activity(f'Checked out item "{item.identifier}".')
        self._emit_event("item_checked_out", item)

        return item

    def return_item(self, identifier: str) -> LendingItem:
        """Return an item and return its updated object."""
        item = self.find_item(identifier)
        item.return_to_library()
        self._record_activity(f'Returned item "{item.identifier}".')
        self._emit_event("item_returned", item)

        return item

    def recent_activity(self) -> tuple[str, ...]:
        """Return an immutable snapshot of recent catalog activity."""
        return tuple(self._recent_activity)
```

Update the package exports.

### `src/library/__init__.py`

```python
"""A small library domain package used in the OOP tutorial modules."""

from library.book import Book
from library.catalog import Library
from library.digital_book import DigitalBook
from library.events import EventDispatchFailure, EventHandler, LibraryEvent, LibraryEventType
from library.lending_item import LendingItem
from library.reports import CatalogReports
from library.search import CatalogPageIterator
from library.snapshots import CatalogSnapshotWriter

__all__ = [
    "Book",
    "CatalogPageIterator",
    "CatalogReports",
    "CatalogSnapshotWriter",
    "DigitalBook",
    "EventDispatchFailure",
    "EventHandler",
    "LendingItem",
    "Library",
    "LibraryEvent",
    "LibraryEventType",
]
```

## The Verification

Run a syntax and import check:

```bash
python -c "from library import Library, LibraryEvent; print(Library); print(LibraryEvent)"
```

Expected output will show class references similar to:

```text
<class 'library.catalog.Library'>
<class 'library.events.LibraryEvent'>
```

---

# Step 4: Register and Remove Callback Functions

## The Target

We are creating:

```text
examples/part_5_callbacks_demo.py
```

This program will subscribe multiple callback functions and demonstrate that callbacks receive events after successful catalog actions.

## The Concept

A callback function must match the event-handler contract:

```python
Callable[[LibraryEvent], None]
```

Read this type from inside out:

- `LibraryEvent` is the input parameter.
- `None` is the return value.
- `Callable[...]` means it can be called like a function.

The callback does not control the catalog. It only receives a notification that an action already happened.

This is an important boundary:

```text
Catalog action succeeds
        ↓
Event is emitted
        ↓
Callback observes the completed action
```

## The Implementation

### `examples/part_5_callbacks_demo.py`

```python
"""Demonstrate first-class callback functions with library events."""

from library import Book, Library, LibraryEvent


def print_event(event: LibraryEvent) -> None:
    """Print every event received from the library."""
    print(
        f"EVENT | type={event.event_type} "
        f"| identifier={event.item_identifier} "
        f'| title="{event.item_title}"'
    )


def record_event_type(
    event_types: list[str],
) -> None:
    """Return nothing; this function is intentionally not a callback.

    It exists to contrast with the closure created in build_event_recorder().
    """
    event_types.append("This function needs an event type argument.")


def build_event_recorder(destination: list[str]):
    """Create a callback that appends event types to destination.

    The inner function closes over destination. It continues to access that
    specific list after build_event_recorder() returns.
    """

    def record_event(event: LibraryEvent) -> None:
        destination.append(event.event_type)

    return record_event


def main() -> None:
    """Subscribe callbacks, emit events, and unsubscribe one callback."""
    library = Library(name="Callback Demo Library")

    received_event_types: list[str] = []
    record_event = build_event_recorder(received_event_types)

    library.subscribe(print_event)
    library.subscribe(record_event)

    library.add_item(
        Book(
            title="Clean Code",
            author="Robert C. Martin",
            isbn="BOOK-001",
            shelf_location="A-01",
        )
    )
    library.check_out_item("BOOK-001")

    print("\nUnsubscribing the terminal-print callback:")
    print(library.unsubscribe(print_event))

    library.return_item("BOOK-001")

    print("\nEvent types recorded by the closure:")
    print(received_event_types)

    print("\nRemoving the same callback again:")
    print(library.unsubscribe(print_event))


if __name__ == "__main__":
    main()
```

## The Verification

Run:

```bash
python examples/part_5_callbacks_demo.py
```

Expected output:

```text
EVENT | type=item_added | identifier=BOOK-001 | title="Clean Code"
EVENT | type=item_checked_out | identifier=BOOK-001 | title="Clean Code"

Unsubscribing the terminal-print callback:
True

Event types recorded by the closure:
['item_added', 'item_checked_out', 'item_returned']

Removing the same callback again:
False
```

The terminal-print callback did not print the return event because it was removed. The closure callback remained subscribed, so it recorded all three event types.

---

# Step 5: Build Configurable Event Handlers with Closures

## The Target

We are creating:

```text
examples/part_5_closures_demo.py
```

This program will create multiple event-handler functions from one factory function.

## The Concept

A **closure** is a function that remembers values from the environment where it was created.

For example:

```python
def make_prefix_printer(prefix: str):
    def print_message(message: str) -> None:
        print(f"{prefix}: {message}")

    return print_message
```

Each returned function remembers a different `prefix`.

```python
info = make_prefix_printer("INFO")
warning = make_prefix_printer("WARNING")

info("Connected")
warning("Connection failed")
```

Output:

```text
INFO: Connected
WARNING: Connection failed
```

Closures are useful when you need a small, configurable function without creating an entire class.

In production code, closures work well for:

- Callbacks
- Predicates
- Small configuration-specific behaviors
- Request hooks
- Retry policies
- Validation rules

## The Implementation

### `examples/part_5_closures_demo.py`

```python
"""Demonstrate configurable event handlers built with closures."""

from collections.abc import Callable

from library import Book, Library, LibraryEvent


def build_event_printer(
    label: str,
    allowed_event_types: set[str],
) -> Callable[[LibraryEvent], None]:
    """Create an event handler with remembered display configuration.

    Args:
        label: Prefix printed before matching events.
        allowed_event_types: Event types this callback should display.

    Returns:
        A callback function that prints only configured event types.
    """
    normalized_label = label.strip()

    if not normalized_label:
        raise ValueError("Event-printer label cannot be blank.")

    normalized_event_types = frozenset(
        event_type.strip()
        for event_type in allowed_event_types
        if event_type.strip()
    )

    if not normalized_event_types:
        raise ValueError("At least one event type must be allowed.")

    def print_matching_event(event: LibraryEvent) -> None:
        """Print an event only when its type belongs to this closure's filter."""
        if event.event_type in normalized_event_types:
            print(
                f'[{normalized_label}] {event.event_type}: '
                f'"{event.item_title}" ({event.item_identifier})'
            )

    return print_matching_event


def main() -> None:
    """Subscribe two differently configured closure callbacks."""
    library = Library(name="Closures Demo Library")

    inventory_handler = build_event_printer(
        label="INVENTORY",
        allowed_event_types={"item_added"},
    )
    lending_handler = build_event_printer(
        label="LENDING",
        allowed_event_types={"item_checked_out", "item_returned"},
    )

    library.subscribe(inventory_handler)
    library.subscribe(lending_handler)

    library.add_item(
        Book(
            title="The Pragmatic Programmer",
            author="David Thomas and Andrew Hunt",
            isbn="BOOK-002",
            shelf_location="A-02",
        )
    )
    library.check_out_item("BOOK-002")
    library.return_item("BOOK-002")


if __name__ == "__main__":
    main()
```

## The Verification

Run:

```bash
python examples/part_5_closures_demo.py
```

Expected output:

```text
[INVENTORY] item_added: "The Pragmatic Programmer" (BOOK-002)
[LENDING] item_checked_out: "The Pragmatic Programmer" (BOOK-002)
[LENDING] item_returned: "The Pragmatic Programmer" (BOOK-002)
```

Each callback remembers its own label and allowed event types. The callbacks were created by the same function but have different behavior because each closure captured different values.

---

# Step 6: Handle Callback Failures Without Breaking the Catalog

## The Target

We are creating:

```text
examples/part_5_callback_failure_demo.py
```

This program verifies that a failing optional callback does not undo a completed catalog operation.

## The Concept

Callbacks are often integration points with code you do not fully control.

For example, a callback may:

- Send data to another service
- Write to a database
- Call a plugin
- Format user-provided data
- Contain a bug

If the callback is optional, the core operation should remain reliable.

Our policy is:

```text
Successful catalog action
    + failing optional callback
    =
Catalog action remains successful
    + callback failure is recorded for inspection
```

This is not the correct policy for every system. It is correct here because event handlers are observers, not required parts of the lending transaction.

## The Implementation

### `examples/part_5_callback_failure_demo.py`

```python
"""Demonstrate error boundaries around optional event callbacks."""

from library import Book, Library, LibraryEvent


def broken_analytics_handler(event: LibraryEvent) -> None:
    """Simulate a failing optional analytics integration."""
    raise RuntimeError(
        f'Analytics service rejected event "{event.event_type}" unexpectedly.'
    )


def main() -> None:
    """Add an item despite an optional callback failure."""
    library = Library(name="Failure Boundary Demo")
    library.subscribe(broken_analytics_handler)

    book = Book(
        title="Clean Architecture",
        author="Robert C. Martin",
        isbn="BOOK-003",
        shelf_location="A-03",
    )

    library.add_item(book)

    print(f'Was "{book.title}" added to the catalog? {book.identifier in library}')
    print(f"Catalog size: {len(library)}")

    failures = library.event_dispatch_failures()

    print(f"\nRecorded callback failures: {len(failures)}")
    for failure in failures:
        print(f"- Handler: {failure.handler_name}")
        print(f"- Event type: {failure.event.event_type}")
        print(f"- Error: {failure.error}")


if __name__ == "__main__":
    main()
```

## The Verification

Run:

```bash
python examples/part_5_callback_failure_demo.py
```

Expected output:

```text
Was "Clean Architecture" added to the catalog? True
Catalog size: 1

Recorded callback failures: 1
- Handler: broken_analytics_handler
- Event type: item_added
- Error: Analytics service rejected event "item_added" unexpectedly.
```

The book was added successfully even though the optional observer failed.

---

# Step 7: Use `*args` and `**kwargs` Safely

## The Target

We are creating:

```text
examples/part_5_variadic_arguments_demo.py
```

This program will show how to accept a flexible number of positional and keyword arguments.

## The Concept

Python provides two special parameter forms:

```python
*args
**kwargs
```

### `*args`

Collects extra positional arguments into a tuple.

```python
def add_numbers(*numbers: int) -> int:
    return sum(numbers)
```

Usage:

```python
add_numbers(1, 2, 3)
```

Inside the function:

```python
numbers == (1, 2, 3)
```

### `**kwargs`

Collects extra named arguments into a dictionary.

```python
def show_options(**options: str) -> None:
    print(options)
```

Usage:

```python
show_options(format="text", locale="en-US")
```

Inside the function:

```python
options == {"format": "text", "locale": "en-US"}
```

These tools are useful when a function genuinely accepts flexible input. They should not be used to hide an unclear API.

For our example, we will create a helper that builds a list of books and allows carefully validated optional metadata.

## The Implementation

### `examples/part_5_variadic_arguments_demo.py`

```python
"""Demonstrate *args and **kwargs with clear validation."""

from typing import Any

from library import Book, Library


def add_books(
    library: Library,
    *books: Book,
    announce: bool = False,
    **metadata: Any,
) -> None:
    """Add one or more books and validate optional metadata.

    Args:
        library: Catalog that should receive the books.
        *books: Any number of Book objects to add.
        announce: Whether to print a message for each added book.
        **metadata: Optional invocation metadata. Only `source` is accepted.

    Raises:
        TypeError: If a supplied value is not a Book or metadata is unknown.
        ValueError: If source is supplied but blank.
    """
    allowed_metadata_keys = {"source"}
    unknown_keys = set(metadata) - allowed_metadata_keys

    if unknown_keys:
        formatted_keys = ", ".join(sorted(unknown_keys))
        raise TypeError(f"Unsupported metadata keys: {formatted_keys}.")

    source = metadata.get("source", "manual entry")

    if not isinstance(source, str):
        raise TypeError("Metadata value 'source' must be a string.")

    cleaned_source = source.strip()

    if not cleaned_source:
        raise ValueError("Metadata value 'source' cannot be blank.")

    for book in books:
        if not isinstance(book, Book):
            raise TypeError("add_books accepts only Book instances.")

        library.add_item(book)

        if announce:
            print(f'Added "{book.title}" from source: {cleaned_source}.')


def main() -> None:
    """Add multiple books with positional and keyword flexibility."""
    library = Library(name="Variadic Arguments Demo")

    clean_code = Book(
        title="Clean Code",
        author="Robert C. Martin",
        isbn="BOOK-001",
        shelf_location="A-01",
    )
    pragmatic_programmer = Book(
        title="The Pragmatic Programmer",
        author="David Thomas and Andrew Hunt",
        isbn="BOOK-002",
        shelf_location="A-02",
    )

    add_books(
        library,
        clean_code,
        pragmatic_programmer,
        announce=True,
        source="initial catalog import",
    )

    print(f"\nCatalog size: {len(library)}")

    try:
        add_books(
            library,
            announce=True,
            unsupported_option="not allowed",
        )
    except TypeError as error:
        print(f"\nExpected error: {error}")


if __name__ == "__main__":
    main()
```

## The Verification

Run:

```bash
python examples/part_5_variadic_arguments_demo.py
```

Expected output:

```text
Added "Clean Code" from source: initial catalog import.
Added "The Pragmatic Programmer" from source: initial catalog import.

Catalog size: 2

Expected error: Unsupported metadata keys: unsupported_option.
```

The function accepts flexible input, but it still validates that flexibility. This is the production-friendly approach: allow extension only where your API explicitly supports it.

---

# Part 5 Reference: First-Class Functions and Flexible Parameters

## Functions Are Objects

Python functions can be assigned to variables:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

handler = greet

print(handler("Ada"))
```

Output:

```text
Hello, Ada!
```

They can also be stored in a list:

```python
handlers = [first_handler, second_handler]
```

Then called later:

```python
for handler in handlers:
    handler(event)
```

This is the mechanism behind the library’s callback registry.

---

## Callable Type Hints

Use `collections.abc.Callable` to describe a function-like value.

```python
from collections.abc import Callable

EventHandler = Callable[[LibraryEvent], None]
```

This means:

```text
A callable that receives one LibraryEvent and returns None.
```

A callable can be:

- A normal function
- A bound instance method
- A callable object with `__call__`
- A closure
- A partial function

---

## Closures

A closure remembers values from its enclosing scope.

```python
def build_multiplier(multiplier: int):
    def multiply(number: int) -> int:
        return number * multiplier

    return multiply
```

Usage:

```python
double = build_multiplier(2)
triple = build_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

Each returned function remembers a distinct `multiplier`.

### Closure Caution: Rebinding Outer Variables

If a nested function needs to reassign—not merely read—an outer variable, use `nonlocal`.

```python
def build_counter():
    count = 0

    def increment() -> int:
        nonlocal count
        count += 1
        return count

    return increment
```

Use `nonlocal` carefully. If a closure accumulates substantial mutable state or grows complex, a class may communicate the design more clearly.

---

## `*args`

`*args` gathers extra positional arguments into a tuple.

```python
def describe_books(*books: Book) -> None:
    for book in books:
        print(book)
```

Usage:

```python
describe_books(first_book, second_book, third_book)
```

You may use a more descriptive name than `args`:

```python
def describe_books(*books: Book) -> None:
    ...
```

This is preferred when the meaning is clear.

---

## `**kwargs`

`**kwargs` gathers extra keyword arguments into a dictionary.

```python
def configure_client(**options: object) -> None:
    print(options)
```

Usage:

```python
configure_client(timeout_seconds=5, retries=3)
```

As with `*args`, choose a meaningful name when possible:

```python
def configure_client(**options: object) -> None:
    ...
```

Avoid accepting arbitrary keyword arguments without validating them. Silent typos are dangerous:

```python
configure_client(timeuot_seconds=5)
```

A strict API should reject unknown keys.

---

## Unpacking with `*` and `**`

The same symbols unpack existing collections when calling functions.

```python
numbers = (1, 2, 3)
print(sum(numbers))
```

For a custom function:

```python
def add_three(first: int, second: int, third: int) -> int:
    return first + second + third

numbers = (1, 2, 3)
print(add_three(*numbers))
```

For named values:

```python
def create_book(title: str, author: str, isbn: str) -> None:
    print(title, author, isbn)

book_data = {
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "978-0132350884",
}

create_book(**book_data)
```

---

## Callback Failure Policy

The callback design in this part catches exceptions from optional event handlers:

```python
try:
    handler(event)
except Exception as error:
    ...
```

This is intentional because a callback is an optional observer.

Do not apply this policy blindly. If a callback performs required work, silently continuing could corrupt the system.

Ask:

1. Is the operation already complete?
2. Is the callback optional or mandatory?
3. Can the failure be retried?
4. Where will failures be reported and monitored?
5. Should the caller receive an error instead?

The correct answer depends on the domain.
