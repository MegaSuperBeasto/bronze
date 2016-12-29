# Bronze
## Collision-resistant ids for distributed systems

<!--

// TODO: create tests
  // spec should be a string and should not contain '-'
  // node clusters should have different PIDs






https://img.shields.io/badge/altusaero-bronze-05a.svg?style=flat-square


https://img.shields.io/npm/dt/bronze.svg?style=flat-square
- npm downloads total

https://img.shields.io/travis/altusaero/bronze.svg?style=flat-square
- travis build status

https://img.shields.io/appveyor/ci/altusaero/bronze.svg?style=flat-square
- appveyor build status

https://img.shields.io/npm/v/bronze.svg?style=flat-square
- npm version

https://img.shields.io/npm/l/bronze.svg?style=flat-square
- npm license

  - https://img.shields.io/github/license/altusaero/bronze.svg?style=flat-square
  - github license

https://img.shields.io/github/issues/altusaero/bronze.svg?style=flat-square
- github issues open

https://img.shields.io/github/issues-pr/altusaero/bronze.svg?style=flat-square
- github pull requests open

https://img.shields.io/github/contributors/altusaero/bronze.svg?style=flat-square
- github contributors



[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)





https://img.shields.io/badge/<SUBJECT>-<STATUS>-<COLOR>.svg

- brightgreen
- green
- yellowgreen
- yellow
- orange
- red
- lightgrey
- blue
- ff69b4
- etc..



-->



```js
const Bronze = require('bronze')

const idGen = new Bronze({name: 'example'})

const id = idGen.generate()
// > 1482810226160-0-14210-example-1a
```


## Installation

```sh
$ npm install bronze --save
```


## Features

- designed for distributed and independent systems
- no time-based (`UUID1`) or random (`UUID4`) collisions
- collision resistant - safely generate up to **9,007,199,254,740,991** ids generated within a single millisecond
- fast - can generate an id in less than .05ms - _even on old hardware_
- can be conveniently sorted by `timestamp` or `name`
- `0BSD` - BSD Zero Clause License


## Specs

A spec determines what goes into an id and how its information is sorted.

`Type 1` - contains a `timestamp` in milliseconds, a `sequence` number, a process id (`PID`), and a `name`
- `1a` is ordered by `timestamp` - `sequence` - `pid` - `name`
  - useful for timestamp-based sorting
  - ex: `1482981306438-0-5132-example-1a`
- `1b` is ordered by `name` - `pid` - `timestamp` - `sequence`
  - useful for name-based sorting
  - ex: `example-5132-1482981317498-0-1b`


## Usage

`Bronze` CLASS
  _static_ _get_ `defaultName` STRING
    - the default name for new instances (if not provided)
  _static_ _get_ `defaultSpec` STRING
    - the default spec for new instances
  _static_ _get_ `specs` OBJECT
    - the default name
  _static_ `parse` (id)
    - parses an id to JSON-compatible object

    - `id` STRING **required**
      - an id to parse

  _new_ Bronze (options)
    `options` OBJECT
      - `sequence` INT
        - the number of ids generated.
        - convenient for continuing creating ids where you left off (no pre-increment required)
        - if invalid falls back to default
        - _default_ = 0
      - `pid` INT
        - a process id to number to use for generated ids
        - if invalid falls back to default
        - _default_ = process.pid, else 0
      - `name` STRING
        - a unique name for the generator - replaces slashes (\\/) with underscores (\_)
        - **IMPORTANT** - do not use the same name at the same time on different machines
        - if invalid falls back to default
        - _default_ = process.env.HOSTNAME, else _constructor_.defaultName
      - `spec` STRING
        - set the spec
        - if invalid falls back to default
        - _default_ = _constructor_.defaultSpec

    _instance_ OBJECT
      `sequence` INT
        - the current sequence
      `pid` INT
        - the pid to
      `name` STRING
        - the parsed name to be used in ids
      `nameRaw` STRING
        - the raw, pre-parsed name
      `spec` INT

      `generate` (options)
        - generates a new id

        `options` OBJECT
          - `json` BOOLEAN
            - returns results as JSON-compatible object
            - _default_ = false

      `nextSequence` ()
        - manually advances the sequence

See [examples](examples).


## Motivation

While developing a distributed system we found the more machines we added the higher risk for collisions while using `UUID1` and `UUID4` to generate unique ids.

The more machines we add, the more chances for collisions, no matter if we used `UUID1` or `UUID4`
  - `UUID1` (timeuuids) can have collisions within 100ns, which can easily happen when migrating/importing data
  - `UUID4` can have collisions at random and while the risk is small it can happen, a chance of data loss does not sit well with us.

### Goals
  - no collisions
    - remove 'randomness' as an element for unique identifiers, which have the possibility of collisions
    - predictability > random/entropy
    - even a slim chance is unacceptable, mission-critical apps
  - keep it simple
  - fast - should be able to uniquely identify data in real-time apps
  - having different sorting/grouping options would be nice
  - a built-in counter (sequence) would be nice for statistics, especially if you can restore it after reboot
  - compatible with Node's `cluster` module with no additional setup
  - eliminate need for performance-draining read-before-writes to check if a unique identifier exists
  - reduced need for counters and auto-incrementing rows


## Notes
  - `name` may contain any character, including dashes, but slashes (\\/) will be replaced with underscores (\_).
    - This allows the opportunity for an id used as a cross-platform filename with little concern.
  - Every machine in your distributed system should use a unique `name`. If every machine has a unique hostname (`process.env.HOSTNAME`) you should be fine.
  - To avoid collisions:
    - Do not reuse a `name` on a different machine within your distributed system's range of clock skew.
    - Be mindful when changing a system's clock - if moving the clock back change temporarily change the `name`
    - Only one instance of Bronze should be created on a single process (`PID`) to avoid collisions.
      - Each worker spawned from Node's `cluster` module receives its own `PID` so no worries here.
  - Sequence counter resets after reaching `Number.MAX_SAFE_INTEGER` (9007199254740991)
  - Without string parsing, timestamp sorting (`spec 1a`) is supported up to `2286-11-20T17:46:39.999Z` (9999999999999)
  - Using with databases:
    - most `text` data types should do the trick


## Future
  - CLI
    ```sh
    $ bronze
    # > returns an id
    $ bronze -n 10
    # > returns 10 ids
    ```

  - Nested IDs
    ```js
    const id1 = idGen.generate({name: 'example'})
    console.log(id1)
    // > 1482810226160-0-14210-example-1a

    // Nested
    idGen.name = id1
    const id2 = idGen.generate()
    console.log(id2)
    // > 1482810226222-1-14210-1482810226160-0-14210-example-1a-1a

    Bronze.parse('1482810226222-1-14210-1482810226160-0-14210-example-1a-1a')
    // { valid: true,
    //   timestamp: 1482810226222,
    //   sequence: 1,
    //   pid: 14210,
    //   name: '1482810226160-0-14210-example-1a',
    //   spec: '1a' }

    // can also nest id2, id3, id4, id5, ...idN
    ```
    - the issue the moment is the possibility of collisions (no unique name)
    - would be pretty cool - imagine the convenience of nesting a video_id into a comment_id