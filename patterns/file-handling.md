# File handling

File handling should preserve source boundaries, validate content carefully, and keep raw source separate from normalized or parsed representations.

For data-oriented tools:

- detect file format intentionally;
- preserve raw source where that matters;
- treat parse failures as first-class results;
- avoid rewriting input merely to satisfy a parser.

For large inputs, prefer streaming or incremental processing when the task allows it.
