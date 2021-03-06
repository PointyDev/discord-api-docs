# API Reference

Discord's API is based around two core layers, a HTTPS/REST API for general operations, and persistent secure websocket based connection for sending and subscribing to real-time events. The most common use case of the Discord API will be providing a service, or access to a platform through the [OAuth2](http://oauth.net/2/) API.

## Base URL

The base URL for all API requests is:

```
https://discordapp.com/api
```

## API Versioning

>danger
>API and Gateway versions below v6 will be discontinued on October 16, 2017, after which they will be non-functioning.

Discord exposes different versions of our API. You can specify version by including it in the request path:

```
https://discordapp.com/api/v{version_number}
```

Omitting the version number from the route will route requests to the current default version. You can find the change log for the newest API version [here](https://discordapp.com/developers/docs/change-log).

>warn
>API and Gateway v6 will be made default on October 16, 2017

###### API Versions

| Version | Status | Default |
| ------- | ------ | ------- |
| 6 | Available | |
| 5 | Deprecated | |
| 4 | Deprecated | |
| 3 | Deprecated | ✓ |

## Authentication

Authenticating with the Discord API can be done in one of two ways:

1. Using a bot token gained by [registering a bot](#DOCS_OAUTH2/registering-applications), for more information on bots see [bots vs user accounts](#DOCS_OAUTH2/bot-vs-user-accounts).
2. Using an OAuth2 bearer token gained through the [OAuth2 API](#DOCS_OAUTH2/oauth2).

For all authentication types, authentication is performed with the `Authorization` HTTP header in the following format:

```
Authorization: TOKEN_TYPE TOKEN
```

### Example Authorization Headers

For bot tokens:

```
Authorization: Bot MTk4NjIyNDgzNDcxOTI1MjQ4.Cl2FMQ.ZnCjm1XVW7vRze4b7Cq4se7kKWs
```

For OAuth bearer tokens:

```
Authorization: Bearer CZhtkLDpNYXgPH9Ml6shqh2OwykChw
```

## Encryption

All HTTP-layer services and protocols (e.g. http, websocket) within the Discord API use TLS 1.2.

## Snowflake IDs

Discord utilizes Twitter's [snowflake](https://github.com/twitter/snowflake/tree/snowflake-2010) format for uniquely identifiable descriptors (IDs). These IDs are guaranteed to be unique across all of Discord, except in some unique scenarios in which child objects share their parent's ID. Because Snowflake IDs are up to 64 bits in size (e.g. a uint64), they are always returned as strings in the HTTP API to prevent integer overflows in some languages. See [Gateway ETF/JSON](#DOCS_GATEWAY/etf-json) for more information regarding Gateway encoding.

## Nullable Resource Fields

Resource fields that may be null have types that are prefixed with a question mark.

## Optional Resource Fields

Resources fields that are optional have names that are suffixed with a question mark.

###### Example Nullable and Optional Field

| Field | Type |
| ----- | ---- |
| optional_field? | string |
| nullable_field | ?string |
| optional_and_nullable_field? | ?string |

## Consistency

Discord operates at a scale where true consistency is impossible. Because of this, lots of operations in our API and in-between our services are [eventually consistent](https://en.wikipedia.org/wiki/Eventual_consistency). Due to this, client actions can never be serialized and may be executed in _any_ order (if executed at all). Along with these constraints, events in Discord may:

- Never be sent to a client
- Be sent _exactly_ one time to the client
- Be sent up to N times per client

Clients should operate on events and results from the API in as much of a idempotent behavior as possible.

## HTTP API

### User Agent

Clients using the HTTP API must provide a valid [User Agent](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.43) which specifies information about the client library and version, in the following format:

```
User-Agent: DiscordBot ($url, $versionNumber)
```

### Rate Limiting

The HTTP API implements a process for limiting and preventing excessive requests in accordance with [RFC 6585](https://tools.ietf.org/html/rfc6585#section-4). API users that regularly hit and ignore rate limits will have their API keys revoked, and be blocked from the platform. For more information on rate limiting of requests, please see the [Rate Limits](#DOCS_RATE_LIMITS/rate-limits) section.

>warn
> A bot account must connect and identify to a [Gateway](#DOCS_GATEWAY/connecting) at least once before being able to send messages.

## Gateway (WebSocket) API

Discord's Gateway API is used for maintaining persistent, stateful websocket connections between your client and our servers. These connections are used for sending and receiving real-time events your client can use to track and update local state. The Gateway API uses secure websocket connections as specified in [RFC 6455](https://tools.ietf.org/html/rfc6455). For information on opening Gateway connections, please see the [Gateway API](#DOCS_GATEWAY/gateways) section.

## Message Formatting

Discord utilizes a subset of markdown for rendering message content on its clients, while also adding some custom functionality to enable things like mentioning users and channels. This functionality uses the following formats:

###### Formats

| Type | Structure | Example |
|---------|-------------|-------------|
| User | <@USER_ID> | <@80351110224678912> |
| User (Nickname) | <@!USER_ID> | <@!80351110224678912> |
| Channel | <#CHANNEL_ID> | <#103735883630395392> |
| Role | <@&ROLE_ID> | <@&165511591545143296> |
| Custom Emoji | <:NAME:ID> | <:mmLol:216154654256398347> |

Using the markdown for either users, roles, or channels will mention the target(s) accordingly.

## Image Formatting

Discord uses ids and hashes to render images in the client. These hashes can be retrieved through various API requests, like [Get User](#DOCS_USER/get-user). Below are the formats, size limitations, and CDN endpoints for images in Discord. The returned format can be changed by changing the extension name at the end of the URL, within the [limitations](#DOCS_REFERENCE/image-formatting-image-formats). The returned size can be changed by appending a querystring of `?size=desired_size` to the URL, within the [limitations](#DOCS_REFERENCE/image-formatting-image-sizes).

###### Image Formats

| Name | Extension |
|-------|------------|
| JPEG | .jpg, .jpeg |
| PNG | .png |
| WebP | .webp |
| GIF | .gif ([user](#DOCS_USER/user-object) avatars only) |

###### Image Sizes

Powers of 2 between 16 and 2048.

>info
>All image URLs are prefixed with https\://cdn.discordapp.com/

###### CDN Endpoints

| Type | Path | Supports |
| ---- | --- | -------- |
| Custom Emoji | emojis/[emoji_id](#DOCS_GUILD/emoji-object).png | PNG |
| Guild Icon | icons/[guild_id](#DOCS_GUILD/guild-object)/[guild_icon](#DOCS_GUILD/guild-object).png | PNG, JPEG, WebP |
| Guild Splash | splashes/[guild_id](#DOCS_GUILD/guild-object)/[guild_splash](#DOCS_GUILD/guild-object).png | PNG, JPEG, WebP |
| Default User Avatar | embed/avatars/[user_discriminator](#DOCS_USER/user-object).png * | PNG |
| User Avatar | avatars/[user_id](#DOCS_USER/user-object)/[user_avatar](#DOCS_USER/user-object).png | PNG, JPEG, WebP, GIF |
| Application Icon | app-icons/[application_id](#MY_APPLICATIONS/top)/[icon](#DOCS_OAUTH2/get-current-application-information).png | PNG, JPEG, WebP |

\* In the case of the Default User Avatar endpoint, the value for `user_discriminator` in the path should be the user's discriminator modulo 5—Test#1337 would be `1337 % 5`, which evaluates to 2.
