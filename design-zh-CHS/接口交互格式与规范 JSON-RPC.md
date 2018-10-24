## 接口交互格式与规范 JSON-RPC

### Request Body

```json
{
  "cmd": "nuls_accounts",
  "params": ["param1", "param2"],
  "min_version": 1.0,
}
```

- 请求参数

| 参数        | 必选  | 类型   | 说明           |
| ----------- | ----- | ------ | -------------- |
| cmd         | true  | string | 执行的 Command |
| params      | true  | array  | 命令参数表     |
| min_version | false | float  | 兼容的最低版本 |

### Response Body

- success

```json
{
  "code":0,
  "msg": "Success",
  "result": {}
}
```

- 响应参数

| 参数   | 必选 | 类型   | 说明                                    |
| :----- | :--- | :----- | --------------------------------------- |
| code   | ture | int    | 请求想要状态，成功返回0。否则返回错误码 |
| msg    | true | string | 用户友好的请求执行结果描述              |
| result | true | object | 方法返回值                              |

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

