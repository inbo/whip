# whip

Whip is a human and machine-readable specification to express **specifications for data**. These specifications are term-based and expressed in YAML, e.g.:

```YAML
dateTerm:
  dateformat: ['%Y-%m-%d', '%Y-%m', '%Y'] # Needs to be ISO8601 format, but don't allow ranges
  mindate: 1830-01-01                     # No dates before 1830
  empty: True                             # Empty values are allowed
```

## License

[MIT License](LICENSE)
