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

We have some Enum Types. We may import it into a validators-modulefile  as:
```
import {
    NUMBER, STRING, STRINGHEX, STRINGHEX32, STRINGHEX64, STRINGHEX128, BOOLEAN, ARRAY, NUMBERORNULL, OBJECT
} from "thingy-schema-validate"
```

This way we may build schemas like this:
```
function1Arguments = {
    argString: STRING
    argNumber: NUMBER
}

function1Response = {
    ok: BOOLEAN
}
```

Then we may build the validationFunctions:
```
import { validate } from "thingy-schema-validate"

validateFunction1Arguments = (args) -> validate(args, function1Arguments)
validateFunction1Response = (response) -> validate(response, function1Response)

export argumentValidators = {
    function1: validateFunction1Arguments
}

export responseValidators = {
    funcion1: validateFunction1Response
}

```

Then we use it to validate:
```
import { argumentValidators } from "validators-module.js"
import { responseValidators } from "validators-module.js"

# e.g. Assuming body-parser.json is active on express 4 
onFunction1 = (req) ->
    try argumentValidators.function1(req.body)
    catch(err) then throw new Error("Function1 invalid Arguments detected!")

    response = executeFunction1(req.body)

    try responseValidators.function1(respnse)
    catch(err) then throw new Error("Function1 invalid Response detected!")
    
    return response

```

---

# Further steps

- check if we need some more types
- maybe improve performance? Fix bugs?


All sorts of inputs are welcome, thanks!

---

# License
[Unlicense JhonnyJason style](https://hackmd.io/nCpLO3gxRlSmKVG3Zxy2hA?view)
