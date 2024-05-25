# Helpers

Camouflage uses handlebars to help you generate dynamic responses if needed. Read more about handlebars [here](https://handlebarsjs.com/guide/).

You can use all the helpers provided by handlebars itself, for example, `if`, `unless`, `each`, and `with`. Camouflage also provides some additional helpers to make some repetitive tasks easy.

### `array` Helper

Usage:

**{{array source='Apple,Banana,Mango,Kiwi' delimiter=','}}**: Generate an array from a source using given delimiter.

### `assign` Helper

Usage: Assign helper can be used to assign a value to a variable, by specifying a name value pair. This can be useful specially when using capture helper using regex and jsonpath selectors. Since running a regex or jsonpath operation is an expensive task, assign helper can be used to capture a value once, store it in a variable and use throughout the mock file. Aesthetically, it also improves readability of the mock file which otherwise would contain long illegible regular expressions repeated throughout the mock file.

Following example shows the usage of a complex combination of helpers, i.e. assign, array, concat, and code.

```javascript
{{assign name='fruits' value=(array source=(concat 'Apple' 'Kiwi' 'Oranges' delimiter='-') delimiter='-')}}
{{#each fruits as |fruit|}}{{#if @last}}{{fruit}}{{else}}{{fruit}}-{{/if}}{{/each}}
```

Explanation:

1. `concat`: We are using `concat` to make a hyphenated string out of 3 seperate string values.
2. `array`: Then we are making an array by splitting the hyphenated string using the hyphen delimiter. Because why not?
3. `assign`: Next, we assign this monstrous redundancy to a variable `fruits`
4. `each`: Next use the helper `each` to loop over the `fruits` array.
5. `if`: Finally, we use `if` helper, to make a beautiful string. We concatenate the fruits with a hyphen (if the current fruit is not the last item in the array).

_Phew, that was quite a journey. We started with 'Apple-Kiwi-Oranges' and we ended it with 'Apple-Kiwi-Oranges'....wait a minute!_

### `concat` Helper

Usage: Concatenates multiple strings together, (static or dynamic), to form a single string.

Example:

- `{{concat 'Camouflage ' 'is ' 'easy!!'}}` results in `Camouflage is easy`.
- You can also pass in a delimiter, i.e. `{{concat '1' '22' '333' delimiter='-'}}` will result in `1-22-333`

### `csv` helper

Usage: CSV Helper allows you to provide a data source as an input along with several combinations of selection policies

- **With a key and value**: Specify the column name with `key` and the value you want to search with `value`. CSV helper returns the first row of the csv file where the value matches the row value in the specified column.
- **Random**: Omitting key and value altogether and specifying `random=true` will fetch you one row at random.
- **All**: Specifying `all=true`, fetches you the entire CSV file, do what you will with the data.

!!! note

    All combinations of the policies return a JSON Array

### `import` Helper

Usage: Import helpers lets you store your reusable templates in shared files, which can then be imported into other files.

Example:

You can create a Camouflage mock file which would contain the response for the request `GET /hello/world`

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "now": "{{import path="/User/john.doe/uselessButReusableNow.mock"}}"
}
```

Here you are importing the file `uselessButResuableNow.mock`, Camouflage helper will replace the import with the contents of your imported file. You can create the reusable file as

```
{{now format='yyyy-MM-dd'}}
```

This seems trivial and of no use but think bigger. Your mock file contains not just one but several responses and you select one of them based on a certain condition. You can break down your mocks into several files and import them as needed, making it easier to maintain.

### `inject` Helper

**Time for the forbidden fruit! The security vulnerability. Everything you have been taught not to do.**

Inject helper allows you replace the hard coded values in your mock files with a javascript code, when you still want to use Camouflage's response builder but you want control over one or two fields.

Example:

```json
{
    "phone": {{#inject}}(()=>{ return Math.floor(1000000000 + Math.random() * 9000000000); })();{{/inject}}
}
```

This translates to a random 10 digit number. _Is it a phone number? Is it not? Who knows!_

!!! caution

    `inject` helper is not enabled by default. If you want to use inject helper, you'd have to enable it when you are creating the helper object.

    ```
    const helpers = new Helpers(true) // setting injectionAllowed = true
    ```

    Or, after creation of helper object using setInjectionAllowed method: `helpers.setInjectionAllowed(true)`,

### `is` Helper

Credits: [danharper/Handlebars-Helpers](https://github.com/danharper/Handlebars-Helpers)

Usage: `is` helper can be considered as an extension of `if` which allows you to evaluate conditions that are lacking in inbuilt helper.

`is` can be used in following three ways:

- With one argument: `is` acts exactly like `if`

  `{{#is x}} ... {{else}} ... {{/is}}`

- With two arguments: is compares the two are equal (a non-strict, == comparison, so 5 == '5' is true)

  `{{#is x y}} ... {{else}} ... {{/is}}`

- With three arguments: the second argument becomes the comparator.

  ```
  {{#is x "not" y}} ... {{else}} ... {{/is}}
  {{#is 5 ">=" 2}} ... {{else}} ... {{/is}}
  ```

Accepted operators are:

- == (same as not providing a comparator)
- !=
- not (alias for !=)
- ===
- !==
- \>
- \>=
- <
- <=
- in (to check if a value exists in an array. ex: {{#is 'John' in (capture from='body' using='jsonpath' selector='$.names')}})

### `now` Helper

Usage:

1. **{{now}}** - Simply using now will give you date in format YYYY-MM-DD hh:mm:ss
2. **{{now format='MM/DD/YYYY'}}** - Format not to your liking? We use luxon to handle date/time, you can provide any format that's supported by luxon. Read more [here](https://moment.github.io/luxon/#/formatting?id=table-of-tokens).
3. **{{now format='epoch'}}** - Time since epoch in milliseconds
4. **{{now format='unix'}}** - Time since epoch in seconds
5. **{{now format='MM/DD/YYYY hh:mm:ss' offset='-10 days'}}** - Use offset to specify the delta for your desired date from the current date.

### `num_between` Helper

Usage:

- **{{num_between lower=500 upper=600}}**: Generate a random number between two values.
- **{{num_between lower=500 upper=600 lognormal=true}}**: Generate random numbers on a bell curve centered between two values.

### `random` Helper

Usage:

- **{{random}}** - Simply using randomValue will generate a 16 character alphanumeric string. ex: _9ZeBvHW5viiYuWRa_.
- **{{random type='ALPHANUMERIC'}}** - You can specify a type as well. Your choices are: 'ALPHANUMERIC', 'ALPHABETIC', 'NUMERIC' and 'UUID'.
- **{{random type='NUMERIC' length=10}}** - Don't want a 16 character output? Use length to specify the length.
- **{{random type='ALPHABETIC' uppercase=true}}** - Finally, specify uppercase as true to get a, well, uppercase string.
