# thingy-schema-validate 

# Background
In the regular communication between thingies, we call the routes similar as functions. There our arguments are all inside one JSON and our response is also one JSON.

However, because we have a gap of whild free network in between where anybody could create any mal-crafted request or response, we should validate these for being well-formed before further processing. 

After some small research about [JSON-schemas](https://json-schema.org/) and [JSON Type Definitions](https://datatracker.ietf.org/doc/rfc8927/), once again the conclusion was drawn that directly building the validators we need seems to be far less of a headache.

# Usage
Requirements
------------
- ESM importability

Installation
------------

Current git version:
```
npm install -g git+https://github.com/JhonnyJason/thingy-schema-validate-output.git
```

Npm Registry
```
npm install -g thingy-schema-validate
```

Current Functionality
---------------------

We have some "Enum" Types. We may import it into a validators-modulefile  as:
```coffeescript
import {
    NUMBER, STRING, STRINGHEX, STRINGHEX32, STRINGHEX64, STRINGHEX128, 
    BOOLEAN, ARRAY, OBJECT, NONNULLOBJECT, NUMBERORNULL, STRINGORNULL, 
    STRINGHEXORNULL, STRINGHEX32ORNULL, STRINGHEX64ORNULL, 
    STRINGHEX128ORNULL, BOOLEANORNULL, ARRAYORNULL
} from "thingy-schema-validate"
```

This way we may build schemas like this:
```coffeescript
functionArgumentSchema = {
    argString: STRING
    argNumber: NUMBER
}
```

Then we may use the validate Function to validate any object for this schema
```coffeescript
import { validate } from "thingy-schema-validate"

args = foundArgsFromAroundTheCorner()

try validate(args, functionArgumentSchema)
catch err then log "seems the args was invalid!"

```

---

# Further steps

- check if we need some more types
- maybe improve performance? Fix bugs?


All sorts of inputs are welcome, thanks!

---

# License
[Unlicense JhonnyJason style](https://hackmd.io/nCpLO3gxRlSmKVG3Zxy2hA?view)
