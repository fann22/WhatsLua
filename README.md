# WhatsLua

UNDER DEVELOPMENT.
But the beta test has been published.

---

# Lua Scripting API

This document describes the Lua API that’s registered by `registerLuaAPI`.  
All of these globals are available inside your Lua scripts.

> **Important Notice**  
> This app is currently in a pre-release state.  
> All Lua API functions described here may change at any time — they may be modified, renamed, or removed without warning.  
> Keep your scripts flexible and check for updates regularly.

# Table of Contents

### Core Context API
- [GetChat](#1.-GetChat%28asJid%29--%3E-string)
- [GetSender](#2.-GetSender%28asJid%29--%3E-string)
- [GetMessageID](#3.-GetMessageID%28%29--%3E-string)
- [GetText](#4.-GetText%28%29--%3E-string)
- [GetArgs](#5.-GetArgs%28%29--%3E-table)

### Messaging API
- [Reply](#6.-Reply%28text%2C-expiration%29)
- [NewMessage](#16.-NewMessage%28%29--%3E-table)
- [SendMessage](#17.-SendMessage%28msgTable%29)

### Logging & Utility API
- [Log](#7.-Log%28msg%29)
- [HasPrefix](#8.-HasPrefix%28str%2C-prefix%29--%3E-bool)
- [HasSuffix](#9.-HasSuffix%28str%2C-suffix%29--%3E-bool)
- [Find](#10.-Find%28str%2C-substr%29--%3E-bool)
- [Split](#14.-Split%28str%2C-separator%29--%3E-table)
- [Join](#15.-Join%28table%2C-joiner%29--%3E-string)
- [Notify](#13.-Notify%28title%2C-body%2C-id%29)

### HTTP API
- [Http_Get](#11.-Http_Get%28url%29--%3E-resTable%2C-errString%7Cnil)
- [Http_Post](#12.-Http_Post%28url%2C-body%29--%3E-resTable%2C-errString%7Cnil)

---

# Lua Scripting API

This document describes the Lua API that’s registered by `registerLuaAPI`.  
All of these globals are available inside your Lua scripts.

%3E **Note**  
> Some functions depend on an internal `currentCtx`.  
> When there’s no active message context (`currentCtx == nil`), they either:
> - return an empty string / empty table, or  
> - do nothing and return.

---

## 1. GetChat(asJid) -> string

Returns the ID of the current chat.

```lua
local chat = GetChat()
local jid  = GetChat(true)
```

### Parameters

| Name  | Type    | Default | Description                          |
| ----- | ------- | ------- | ------------------------------------ |
| asJid | boolean | `false` | If `true`, return JID instead of LID |

### Behavior

* If `asJid == true`: returns `currentCtx.ChatJID`
* Otherwise: returns `currentCtx.ChatLID`
* If there’s no active context: returns `""`

---

## 2. GetSender(asJid) -> string

Returns the sender ID of the current message.

```lua
local senderLid = GetSender()
local senderJid = GetSender(true)
```

### Parameters

| Name  | Type    | Default | Description                          |
| ----- | ------- | ------- | ------------------------------------ |
| asJid | boolean | `false` | If `true`, return JID instead of LID |

### Behavior

* If `asJid == true`: returns `currentCtx.SenderJID`
* Otherwise: returns `currentCtx.SenderLID`
* If there’s no active context: returns `""`

---

## 3. GetMessageID() -> string

Returns the ID of the current message.

```lua
local msgId = GetMessageID()
```

### Behavior

* Returns `currentCtx.MessageID`
* If there’s no active context: returns `""`

---

## 4. GetText() -> string

Returns the text content of the current message.

```lua
local text = GetText()
```

### Behavior

* Returns `currentCtx.Text`
* If there’s no active context: returns `""`

---

## 5. GetArgs() -> table<string>

Returns the parsed arguments of the current message as a Lua array-style table.

```lua
local args = GetArgs()
local firstArg = args[1]
```

### Behavior

* Builds a new table and appends each value from `currentCtx.Args`
* If there’s no active context: returns an empty table (`{}`)

---

## 6. Reply(text, expiration)

Sends a reply to the current message.

```lua
Reply("Hello there!")                 -- with expiration (default: true)
Reply("This message persists", false) -- no expiration
```

### Parameters

| Name       | Type    | Default | Description                         |
| ---------- | ------- | ------- | ----------------------------------- |
| text       | string  | —       | Reply text (required)               |
| expiration | boolean | `true`  | If `true`, send as expiring message |

### Behavior

* If `currentCtx` is `nil`, nothing is sent.
* Internally calls `sendReply(currentCtx, text, expiration)`.

---

## 7. Log(msg)

Sends a log message to the Go-side logger.

```lua
Log("Something happened in Lua")
```

### Parameters

| Name | Type   | Description         |
| ---- | ------ | ------------------- |
| msg  | string | Log message content |

### Behavior

* Wraps the message with `"[LUA] "` and forwards it to `reportError`.

---

## 8. HasPrefix(str, prefix) -> bool

Checks if a string starts with a given prefix.

```lua
if HasPrefix("!ping", "!") then
  Reply("This looks like a command")
end
```

### Parameters

| Name   | Type   | Description         |
| ------ | ------ | ------------------- |
| str    | string | Input string        |
| prefix | string | Prefix to check for |

### Returns

* `true` if `str` starts with `prefix`
* `false` otherwise

---

## 9. HasSuffix(str, suffix) -> bool

Checks if a string ends with a given suffix.

```lua
if HasSuffix(filename, ".txt") then
  Reply("Text file detected")
end
```

### Parameters

| Name   | Type   | Description         |
| ------ | ------ | ------------------- |
| str    | string | Input string        |
| suffix | string | Suffix to check for |

### Returns

* `true` if `str` ends with `suffix`
* `false` otherwise

---

## 10. Find(str, substr) -> bool

Checks if a substring is contained inside a string.

```lua
if Find(GetText(), "hello") then
  Reply("Hi there!")
end
```

### Parameters

| Name   | Type   | Description             |
| ------ | ------ | ----------------------- |
| str    | string | Input string            |
| substr | string | Substring to search for |

### Returns

* `true` if `substr` is contained in `str`
* `false` otherwise

---

## 11. Http_Get(url) -> resTable, errString|nil

Performs an HTTP GET request with a 10-second timeout.

```lua
local res, err = Http_Get("https://httpbin.org/get")

if err ~= nil then
  Log("Http_Get failed: " .. err)
  return
end

if res.status ~= 200 then
  Log("Unexpected status: " .. res.status)
end

Reply("Raw body: " .. res.body, false)
```

### Parameters

| Name | Type   | Description |
| ---- | ------ | ----------- |
| url  | string | Request URL |

### Returns

* `resTable` – a table with detailed response info (see [HTTP response table layout](#http-response-table-layout)), or `nil` on error.
* `errString` – error message string if something fails, otherwise `nil`.

---

## 12. Http_Post(url, body) -> resTable, errString|nil

Performs an HTTP POST request with content-type `application/json` and a 10-second timeout.

```lua
local payload = '{"name":"lua-bot"}'
local res, err = Http_Post("https://httpbin.org/post", payload)

if err ~= nil then
  Log("Http_Post failed: " .. err)
  return
end

Reply("Status: " .. res.status, false)
```

### Parameters

| Name | Type   | Description                                          |
| ---- | ------ | ---------------------------------------------------- |
| url  | string | Request URL                                          |
| body | string | Request body. Sent as `application/json` by default. |

### Returns

Same as `Http_Get`:

* `resTable` – response table (see below), or `nil` on error.
* `errString` – error message string or `nil` on success.

---

## HTTP response table layout

Both `Http_Get` and `Http_Post` return the same structure when they succeed:

```lua
-- Example shape:
-- {
--   status  = <number>,
--   body    = <string>,
--   headers = { [headerName] = string or {string, ...} },
--   json    = <Lua value> -- only if body is valid JSON
-- }
```

### Fields

#### `status` (number)

HTTP status code.

```lua
if res.status == 200 then
  -- ok
elseif res.status == 404 then
  -- not found
end
```

#### `body` (string)

Raw response body as a string.

```lua
local text = res.body
```

#### `headers` (table)

All HTTP response headers.

* If a header has a single value → it’s a string.
* If a header has multiple values → it’s an array-style table of strings.

Example:

```lua
local headers = res.headers
local contentType = headers["Content-Type"]

-- If server sends multiple values:
local setCookie = headers["Set-Cookie"]
if type(setCookie) == "table" then
  local firstCookie = setCookie[1]
end
```

#### `json` (Lua value, optional)

If the body is valid JSON, the result is parsed and stored in `res.json`.
If parsing fails, `res.json` is not set.

JSON is converted like this:

* JSON object → Lua table with string keys
* JSON array → Lua table with numeric indices starting at 1
* JSON string → Lua string
* JSON number → Lua number
* JSON boolean → Lua boolean
* JSON `null` → `nil`

Example:

```lua
local res, err = Http_Get("https://api.example.com/users")
if err ~= nil then
  Log("API error: " .. err)
  return
end

if res.json ~= nil then
  local data = res.json

  -- assuming body was:
  -- { "users": [ { "name": "Alice" }, { "name": "Bob" } ] }
  local users = data.users
  if users and users[1] then
    local firstName = users[1].name
    Reply("First user: " .. firstName, false)
  end
else
  Log("Response is not valid JSON")
end
```

On any network / IO error or JSON read error:

* `resTable` = `nil`
* `errString` = error message

---

## 13. Notify(title, body, id)

Triggers a notification on the host side.

```lua
Notify("New event", "Someone used the bot", "event-123")
Notify("Ping")          -- only title
Notify("", "Body only") -- only body
```

### Parameters

All parameters are optional, but **at least one must be non-empty**.

| Name  | Type   | Default | Description                       |
| ----- | ------ | ------- | --------------------------------- |
| title | string | `""`    | Notification title                |
| body  | string | `""`    | Notification body/message         |
| id    | string | `""`    | Optional identifier for the notif |

### Behavior

* If `title`, `body`, and `id` are **all empty**:

  * `Notify` raises an error: `Notify; You have to set atleast 1 parameter.`
* Otherwise calls `Notify(title, body, id)` on the Go side.

---

## 14. Split(str, separator) -> table<string>

Splits a string into parts.

```lua
local parts = Split("a b c")
-- parts[1] = "a", parts[2] = "b", parts[3] = "c"

local csv = Split("a,b,c", ",")
```

### Parameters

| Name      | Type   | Default | Description                            |
| --------- | ------ | ------- | -------------------------------------- |
| str       | string | —       | Input string                           |
| separator | string | `" "`   | Separator to split on (default: space) |

### Returns

* A new table where each element is a substring from `strings.Split`.

---

## 15. Join(table, joiner) -> string

Joins a table of values into a single string.

```lua
local words = {"hello", "world"}
local msg = Join(words)          -- "hello world"
local csv = Join(words, ",")     -- "hello,world"
```

### Parameters

| Name   | Type   | Default | Description                    |
| ------ | ------ | ------- | ------------------------------ |
| table  | table  | —       | Lua table to join (required)   |
| joiner | string | `" "`   | String to put between elements |

### Behavior

* Raises an error if `table` is `nil`:

  * `Join; String table is empty`
* Converts each value to string (`v2.String()`) before joining.
* Returns a single string.

---

## 16. NewMessage() -> table

Creates a new table representing an outgoing message.

```lua
local msg = NewMessage()
msg.chat = GetChat(true)
msg.text = "Hi from Lua"
msg.quote = true
msg.expiration = 3600  -- seconds
```

### Returns

* A new message table.
  The fields are interpreted later by `SendMessage`.

---

## 17. SendMessage(msgTable)

Sends a message described by `msgTable`.

```lua
local msg = NewMessage()
msg.chat = GetChat(true)   -- required
msg.text = "Hello there!"  -- required
msg.quote = true           -- optional
msg.expiration = 600       -- optional (seconds)

SendMessage(msg)
```

### Expected Fields in `msgTable`

| Field      | Type    | Required | Description                          |
| ---------- | ------- | -------- | ------------------------------------ |
| chat       | string  | Yes      | Target chat identifier               |
| text       | string  | Yes      | Message text                         |
| quote      | boolean | No       | If `true`, quote the current message |
| expiration | number  | No       | Message expiration time in seconds   |

> The actual sending is handled by the Go-side `sendMessage` implementation.

---

## Quick Example

A simple command that replies with info about the sender and chat:

```lua
if HasPrefix(GetText(), "!whoami") then
  local sender = GetSender(true)
  local chat   = GetChat(true)

  local msg = "You are: " .. sender .. "\nChat: " .. chat
  Reply(msg, false)
end
```

---

## License

WhatsLua © 2025 Fann22  
Released under the **GNU General Public License v3 (GPL-3.0)**.  
See the `LICENSE` file for more information.
