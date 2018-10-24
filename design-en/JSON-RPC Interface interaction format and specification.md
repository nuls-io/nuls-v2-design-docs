## JSON-RPC Interface interaction format and specification

### Request Body

```json
{
  "cmd": "nuls_accounts",
  "params": ["param1", "param2"],
  "min_version": 1.0,
}
```

- parameters

| parameter   | required | type   | description                                  |
| ----------- | -------- | ------ | -------------------------------------------- |
| cmd         | true     | string | The command to call                          |
| params      | true     | array  | parameters                                   |
| min_version | false    | float  | The minimum version of the interface to call |

### Response Body

- success

```json
{
  "code":0,
  "msg": "Success",
  "result": {}
}
```

- returns

| parameter | required | type   | description                                                  |
| :-------- | :------- | :----- | ------------------------------------------------------------ |
| code      | ture     | int    | The result status of the request, 0 means success. Otherwise return an error code |
| msg       | true     | string | User-friendly request execution result description           |
| result    | true     | object | Method return value                                          |

### Error Code

#### JSON RPC Standard errors

| Code      | Possible Return message | Description                                                  |
| --------- | ----------------------- | ------------------------------------------------------------ |
| 0         | Success                 | Operation success                                            |
| 1         | Parse error             | Invalid JSON was received by the server. An error occurred on the server while parsing the JSON text. |
| 2         | Invalid Request         | The JSON sent is not a valid Request object.                 |
| 3         | Method not found        | The method does not exist / is not available.                |
| 4         | Invalid params          | Invalid method parameter(s).                                 |
| 5         | Internal error          | Internal JSON-RPC error.                                     |
| 6         | Unauthorized            | Should be used when some action is not authorized, e.g. sending from a locked account. |
| 7         | Action not allowed      | Should be used when some action is not allowed, e.g. preventing an action, while another depending action is processing on, like sending again when a confirmation popup is shown to the user (?). |
| 8         | Timeout                 | Should be used when an action timedout.                      |
| 9         | Conflict                | Should be used when an action conflicts with another (ongoing?) action. |
| 10        | Execution error         | Will contain a subset of custom errors in the data field. See below. |
| 11 to 100 | `Server error`          | Reserved for implementation-defined server-errors.           |

#### Custom error fields

Custom error `10` can contain custom error(s) to further explain what went wrong.

```js
{
    code: 10,
    msg: 'Execution error',
}
```

