# Vue Moola
A vue 2 component for a money mask input.

## How to initialize Vue Moola
Vue moola is built as a vue plugin. It can be initialized just as the Vue documentation states.

```javascript
import Moola from "@pderas/vue2-moola";

Vue.use(Moola);
```
## Usage

#### Creation
Vue moola is easily created, and has many properties that can be changed.
```HTML
<moola value="0"></moola>
```

## Properties
| Property  | Required | Type                 | Default                 | Description                                   |
|-----------|----------|----------------------|-------------------------|-----------------------------------------------|
| delimiter | false    | String               | ','                     | A character to separate the numbers thousands by. (e.i. '1,000') |
| max       | false    | Number               | Number.MAX_SAFE_INTEGER | The max value for the input                   |
| min       | false    | Number               | Number.MIN_SAFE_INTEGER | The min value for the input                   |
| nullable | false | Boolean | false | Sets the default empty value to either 0 or null |
| precision | false    | Number               | 2                       | Amount of decimals to allow                   |
| prefix    | false    | String               | '$'                     | A prefix for the input (can be set to null)   |
| suffix    | false    | String               | null                    | A suffix for the input (e.i. '%')             |
| value     | false    | Number&#124;String   | 0                       | Value for the input, can be used with v-model |



## License
This project is covered under the MIT License. Feel free to use it wherever you like.