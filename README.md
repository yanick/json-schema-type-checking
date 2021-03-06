# json-schema-function-signatures 

This package provides a way to validate the inputs, output, and context 
of functions via [JSON schema](https://json-schema.org) definitions.

## Basic usage

Using the proposed pipeline operator via [babel](https://www.npmjs.com/package/@babel/plugin-proposal-pipeline-operator).


    import { number, object, string } from "json-schema-shorthand";

    import { funcSchema } 
        from "json-schema-function-signatures";

    const myFunc =
        function(foo, bar) {
            return { sum: foo + bar, ctx: { baz: this } };
        }
            |> funcSchema(
                [number({ minimum: 2 }), number({ maximum: 3 })],
                object({                                          
                    sum: number({ minimum: 10 }),
                    ctx: object({ baz: string({ minLength: 3 }) })
                }),
                string()
            );


Same thing, but using [lodash](https://github.com/lodash)'s `_.flow`.

    import { number, object, string } from "json-schema-shorthand";

    import { funcSchema } from "json-schema-function-signatures";

    import _ from 'lodash';

    const myFunc = _.flow([
        function(foo, bar) {
            return { sum: foo + bar, ctx: { baz: this } };
        },
        funcSchema(
            [number({ minimum: 2 }), number({ maximum: 3 })],
            object({
                sum: number({ minimum: 10 }),
                ctx: object({ baz: string({ minLength: 3 }) })
            }),
            contextSchema(string()
        )
    ]);

Still same thing, barebone without any helper function.

    import { number, object, string } from "json-schema-shorthand";

    import { funcSchema } from "json-schema-function-signatures";

    const myFunc = funcSchema(
        [number({ minimum: 2 }), number({ maximum: 3 })],
        object({
            sum: number({ minimum: 10 }),
            ctx: object({ baz: string({ minLength: 3 }) })
        }),
        contextSchema(string()
    )(
        function(foo, bar) {
            return { sum: foo + bar, ctx: { baz: this } };
        },
    );


## Exports

### inSchema( schema )( targetFunction )

Validates the arguments of the target function against the `schema`. 

If `schema` is an array, it's automatically inflated as the `items` of 
an `array` type. I.e.,

    const myFunc = ( foo => { ... } )
        |> inSchema([{type => 'string'}]);

    // equivalent to 

    import { array } from 'json-schema-shorthand';
    const myFunc = ( foo => { ... } )
        |> inschema({ type => 'array', items => [{type => 'string'}] });


### outSchema( schema )( targetFunction )

Validates the return value of the target function against the `schema`. 

### contextSchema( schema )( targetFunction )

Validates the context of the target function against the `schema`. 

### funcSchema( inSchema, outSchema, contextSchema )( targetFunction )

Shortcut to call one or more of the `in`/`out`/`context`Schema wrappers.
Passing `undefined` as one schema disables that validation.

### FunctionSignatures 

    import FunctionSignatures from 'json-schema-function-signatures';

    const myValidator = new FunctionSignatures();

    const myFunc = function(x,y) { return x+y }
        |> myValidator.outSchema({ type: 'number', minimum: 12 });

The default export of the package is the `FunctionSignatures` class.

#### new FunctionSignatures(options) 

    const myValidator = new FunctionSignatures({
        ajv:      new Ajv(),
        onError:  error => { ... },
        disabled: false,
    )};

Accepts the following options:

1. *ajv*: custom [Ajv](https://github.com/epoberezkin/ajv) 
object for the schema validations.

1. *onError*: callback invoked when a validation fails. Will be passed
    an error object encapsulating the validation errors. Default to throwing an
   exception.

1. *disabled*: if set to true, no validation is performed.

### disabled 

Getter/setter for the `disabled` attribute of the object. If `true`, no
validation is performed.

