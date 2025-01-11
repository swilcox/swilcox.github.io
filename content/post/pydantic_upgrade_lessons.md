+++
title = 'Pydantic Upgrade Lessons'
date = 2025-01-11T09:45:57-06:00
draft = false
summary = 'Some things I picked up while doing a Pydantic 1.x to 2.x upgrade'
tags = ['python', 'pydantic']
+++

# Pydantic Upgrade Lessons

## Background

Like many projects of any size, moving from Pydantic 1.x to 2.x is not a trivial matter. As part of learning a new codebase, I decided to take on that conversion, knowing I'd get exposed to lots of models and get a better feel for the project.

## Tips

* Read over the [Pydantic documentation / guide for conversion](https://docs.pydantic.dev/latest/migration/).
* Consider using the `bump-pydantic` tool mentioned in the guide. While manual conversion might be preferred in some situations, this tool can significantly speed up the more repetitive tasks.
* Pydantic 2.x provides a more consistent and validation-focused implementation of models, with enhanced type checking and stricter validation rules. [More information on vision here](https://pydantic.dev/articles/pydantic-v2).
* Be aware that the enhanced validation and type checking in 2.x may come with some performance tradeoffs ([see performance discussion here](https://github.com/pydantic/pydantic/discussions/6748)).

### 1.x to 2.x converter

As mentioned in the migration documentation, there is a tool called [bump-pydantic](https://github.com/pydantic/bump-pydantic) that can help in your migration. While it doesn't handle 100% of the conversion, it significantly accelerates the process. In my experience, it errs on the side of caution and makes only the most straightforward changes. For example, I found many instances where I needed to replace `json()` calls with `model_dump_json()` calls, especially when the code wasn't directly referencing a specific Pydantic model.

### YAML plugin

For codebases using YAML, particularly with the [pydantic-yaml](https://pypi.org/project/pydantic-yaml/) library, there's a significant change in implementation: instead of subclassing `BaseModel` from Pydantic, you now use the `parse_yaml_raw_as()` method from the library (in our case).

## Old vs New Schema Problems

One of the initial goals for this migration was to minimize outward-facing differences, including maintaining consistency in the generated `openapi-spec.json`.

### Optional Values and null

This is where subtle but significant differences emerged. In our previous code, we often had situations like this:

```python
class ASimpleModel(BaseModel):
    a_required_field: str
    an_optional_field: Optional[str]
```

While this syntax correctly expresses the intent, Pydantic 2.x requires more precise type definitions. The more precise way to express this in Pydantic 2.x is:

```python
class ASimpleModel(BaseModel):
    a_required_field: str
    an_optional_field: str | None = None  # or Optional[str] = None    
```

In Pydantic 2.x, optional fields must explicitly specify a default value via assignment, whereas in 1.x, `None` was the implicit default value.

### Stealth optional null values

In Pydantic 1.x, OpenAPI spec files would often omit `null` entirely for optional fields where `null` was the default. If you want to maintain this behavior in 2.x while still setting a default value to `null`, you can use this approach:

```python
from pydantic.json_schema import SkipJsonSchema

class ASimpleModel(BaseModel):
    a_required_field: str
    an_optional_field: str | SkipJsonSchema[None] = None
```

This approach satisfies mypy's type checking requirements while controlling how the field appears in the OpenAPI spec. For more detailed information about handling null values, [see this comprehensive article](https://skaaptjop.medium.com/how-i-use-pydantic-unrequired-fields-so-that-the-schema-works-0010d8758072).

### Post JSON Schema generation tweaking

Pydantic 2.x handles nullable fields differently in the generated OpenAPI spec, placing null as a choice within an `anyOf` block. To maintain compatibility with systems expecting the older `nullable=true` attribute, I created a script to transform the generated schema:

```python
from typing import Any


def convert_anyof_to_nullable(schema: dict[Any, Any] | list[Any]) -> None:
    """
    Recursively convert anyOf with null to nullable: true in OpenAPI schema

    NOTE: this *shouldn't* be necessary but is a workaround to keep generation of the schema closer to what was
    happening with Pydantic 1.x. This is a temporary solution until we verify that other systems can handle the
    more "compliant" schema that Pydantic 2.x generates.

    While we're at it... this is also adding the description "An enumeration." to enums that don't have a description.
    This is purely to match the schema that Pydantic 1.x was generating and to keep our diffs small.
    """
    if isinstance(schema, dict):
        if "anyOf" in schema:
            types = [t.get("type") for t in schema["anyOf"]]
            if "null" in types and len(types) == 2:
                # Find the non-null type definition
                non_null_schema = next(t for t in schema["anyOf"] if t.get("type") != "null")

                # Preserve all attributes from the parent schema except 'anyOf'
                parent_attrs = {k: v for k, v in schema.items() if k != "anyOf"}

                # Clear the current schema
                schema.clear()

                # Update with the non-null schema attributes
                if "$ref" in non_null_schema:
                    schema["allOf"] = [non_null_schema]
                else:
                    schema.update(non_null_schema)

                # Restore parent attributes (they take precedence over non_null_schema)
                schema.update(parent_attrs)

                # Add nullable flag
                schema["nullable"] = True

        # fix for enums
        if "enum" in schema and "description" not in schema:
            schema["description"] = "An enumeration."

        # Recursively process all dictionary values
        for value in schema.values():
            if isinstance(value, (dict, list)):
                convert_anyof_to_nullable(value)

    elif isinstance(schema, list):
        # Recursively process all list items
        for item in schema:
            if isinstance(item, (dict, list)):
                convert_anyof_to_nullable(item)
```

This transformation converts schemas from:

```json
{
    "type": "object",
    "properties": {
        "foo": {
            "anyOf": [
                {"type": "string"},
                {"type": "null"}
            ]
        }
    }
}
```

to:

```json
{
    "type": "object",
    "properties": {
        "foo": {
            "type": "string",
            "nullable": true
        }
    }
}
```

To apply this transformation in your FastAPI application, add this code after defining your app, preferably in a [lifetime event definition](https://fastapi.tiangolo.com/advanced/events/):

```python
openapi_schema = app.openapi()
convert_anyof_to_nullable(openapi_schema)
app.openapi_schema = openapi_schema
```

## Conclusion

While the migration to Pydantic 2.x requires careful attention to detail, it brings improved type safety and validation capabilities. The migration process can be streamlined using available tools, though some manual intervention is usually necessary. The most challenging aspects often involve maintaining backward compatibility with existing systems while adopting the new, more precise type system.
