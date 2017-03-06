# whip

Whip is a human and machine-readable syntax to express **specifications for data**. It can be used as a whip to test how well data meets certain specifications, be it a feather :sweat_smile: or a chain whip :scream:.

Example:

```yaml
my_date_field:
  dateformat: ['%Y-%m-%d', '%Y-%m', '%Y'] # Needs to be ISO8601 format, but don't allow ranges
  mindate: 1830-01-01                     # No dates before 1830
  empty: True                             # Empty values are allowed
```

## Documentation

* [Syntax](docs/syntax.md)

## Contributors

* [Peter Desmet](https://github.com/peterdesmet)
* [Stijn Van Hoey](https://github.com/stijnvanhoey)
* [Christian Gendreau](https://github.com/cgendreau)
* [Dimitri Brosens](https://github.com/DimEvil)
* [John Wieczorek](https://github.com/tucotuco)

## License

[MIT License](LICENSE)
