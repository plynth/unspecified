===========
unspecified
===========

Does it look like a duck?

Example::

    >>> from uuid import uuid4
    >>> from unspecified import unspecified
    >>> value = {"id": uuid4()}
    >>> assert value == {"id": unspecified.uuid}
    True

``unspecified`` is a Python library for testing if a value has specific
characteristics without actually caring about the value itself. It is
particularly useful for comparing dictionaries or lists of results
against expected values without having to know the actual values.

The following is a typical test::

    def test_create_duck():
        """
        Verify a duck can be created
        """
        value = create_duck(name="Fred")
        assert value["id"]
        assert value["created"]
        assert value["name"] == "Fred"

The above test is only verifying particular fields in the resulting 
``dict``. Tests in general should verify all resulting values and that 
those values exist and meet expectations. This is good practice because 
with refactoring, fields may be added or removed that accidentally break
the contract the function has.

The complication here is that there are several generated  values that 
can't be known ahead of time. Monkey patching can be used to mock a 
time or a specific generated ID, but you end up testing the mock instead
of the actual functionality.

`unspecified` can be used to contract test values that are expected to
have a certain type or format, but whose value need not or can not be 
known ahead of time.

Contract testing can be done manually. If the function returned a single 
value, it is relatively easy 
to compare::

    assert isinstance(value, UUID)
    
However when many values in a list or dictionary are returned, comparison
is much harder::
    

    def test_create_duck():
        """
        Verify a duck can be created
        """
        now = datetime.now()
        value = create_duck(name="Fred")
        # Verify there are only the expected keys
        assert set(value.keys()) == {"id", "created", "name"}
        assert isinstance(value["id"], UUID)
        assert (
            isinstance(value["created"], datetime)
            and value["created"] >= now
            and value["created"] <= now + timedelta(minutes=1)
        )
        assert value["name"] == "Fred"

``unspecified`` simplifies contract testing, especially for complex types
like ``dict``s and ``object``s::


    def test_create_duck():
        """
        Verify a duck can be created
        """
        value = create_duck(name="Fred")
        # Verify there are only the expected keys
        assert value == {
            "id": unspecified.uuid,
            "created": unspecified.datetime.approximately.now,
            "name": "Fred",
        }



``unspecified`` works well with most Python standard library types. It also
works well with JSON::

    r = request.get(
        "http://localhost/api/ducks",
        headers={"Accept": "application/json"}
    )
    assert r.data == unspecified.json.dict(
        total=2,
        items=[
            dict(
                id=unspecified.uuid.hex,
                created=unspecified.datetime.rfc3339,
                name="Fred",
            ),
            dict(
                id=unspecified.uuid.hex,
                created=unspecified.datetime.rfc3339,
                name="Lucile",
            ),
        ],
    )

With JSON, complex Python types are often converted to a string making
type comparison difficult. ``unspecified`` has you covered. Types can 
be chained together to represent type encapsulation::

    assert "1" == unspecified.str.int
    assert "1.33" == unspecified.str.float
    assert "f84abed8-34f1-4aeb-b081-cdcbe3522738" == unspecified.uuid.str
    assert "f84abed834f14aebb081cdcbe3522738" == unspecified.uuid.hex
    assert "1990-12-31T23:59:60Z" == unspecified.datetime.rfc3339


``unspecified`` also works well with 
[SpringField](https://github.com/plynth/springfield)::

    assert value == unspecified.Entity(
        DuckEntity, 
        id=unspecified.uuid,
        created=unspecified.datetime.approximately.now,
        name="Fred"
    )
