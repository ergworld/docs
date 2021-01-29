# ErgWorld (beta)

Documentation to integrate custom applications into the new ErgWorld platform

## How to register an application to use ErgWorld's services
Currently, this process is manual. Please drop us an [email](mailto:support@ergworld.com?subject=Application%20Request) with the following details:
 - Application name
 - Application description - please mention if you want to send us data or just want to display a participation board
 - contact email

We will review and approve the application and will respond to your **application email** with the following credentials:
- `client_id` - application identifier
- `client_secret` - application secret - lives for 30 days only!
- `scope` - `observe` or `publish` depending on your application's needs

## How to connect to ErgWorld 
Once you received the credentails, you are able to use the platform.

Estabilising a connection to ErgWorld is a 2-step process:
1. request access to establish connection
2. estabilish WebSocket connection with the provided access token

#### 1. Requesting access

When requesting access, you will also need to specify a `uid` parameter that's unique to the end-user/ergometer and a `scope` (defaults to `observe`) parameter

If the scope is `observe`, you will only be allowed to un/subscribe.

Request
```
POST https://auth.ergworld.com/token HTTP/1.1
Content-type: application/json

{
	"client_id": "[client_id]",
	"client_secret": "[client_secret]",
	"uid": "1234567890",
	"scope": "publish"
}
```

Response
```
{
  "url": "[websocket_url_with_access_token]",
  "scope": "publish"
}
```

#### 2. Estabilish WebSocket connection to ErgWorld

The `url` property contains the full URL to establish a WebSocket connection with.

> **Important** The access tokens are valid for 60 seconds only!

## ErgWorld Message Format

Subscriptions, metadata, workout and progress updates happen via WebSocket messages in a **stringified JSON** format.

Stringified JSON is a list of `action` and `payload`: `[ action, payload ]`.

String values (such as the identifier or the indexed metadata fields) have a maximum length of 64, anything longer will be truncated.

Example in JavaScript:
```ts
const wsClient: WebSocket = new WebSocket(url); // url received

wsClient.onopen = function (): void {
  // example subscription
  const message: string = JSON.stringify( [ 0xa, { group: "my_group" } ] );
  ws.send(message);
};
```

The message you will receive is either a number for heat countdown or a list of participations:

```js
[ // list of participations
  [ // a participation
    "[unique_id]", // end-user/ergometer identifier
    1603664079211, // joined timestamp
    "g=[group]&l=[lobby]&t=[type]&n=[name], // metadata
    [
      0, // ElapsedTime
      0, // Distance
      0, // Pace
      0, // Watts
      0, // Cal/Hr
      0, // Stroke Rate
      0, // Heart Rate
      0, // Workout State
      0, // Workout Type
      0, // Workout Duration
      0, // Workout Duration Type
      0, // Interval Count
      0, // Projected Time
      0, // Projected Distance
    ],
    [
      0, // Longitude
      0, // Latitude
    ]
  ],
  ...
]
```

## Subscribe to ErgWorld

ErgWorld allows subscriptions to the indexed metadata fields of individual participations.

List of currently supported indexed metadata fields:
- `g` - (optional - defaults to `'*'`) `string` unique group ID
- `l` - (optional - defaults to `'*'`) `string` lobby within group
- `t` - (optional) `string` we intended tis field for a short manufacturer and model specs
- `n` - (optional) `string` participation display name

If the fields aren't set, they won't appear in the broadcasted state.

#### Subscribe

> Action: `0xa`

In order to start receiving updates, a subscription is needed.

Example - global subscription:
```js
wsClient.send(JSON.strigify(
  [ 0xa ]
));
```

Same as
```js
wsClient.send(JSON.strigify(
  [
    0xa,
    {
      group: '*',
      lobby: '*',
    }
  ]
));
```

Example - global subscription for a lobby:
```js
wsClient.send(JSON.strigify(
  [
    0xa,
    {
      lobby: 'hakuna matata'
    }
  ]
));
```

Example - group:
```js
wsClient.send(JSON.strigify(
  [
    0xa,
    {
      group: 'most_epic_group_ever'
    }
  ]
));
```

Example - group lobbies:
```js
wsClient.send(JSON.strigify(
  [
    0xa,
    {
      group: 'most_epic_group_ever',
      lobby: 'hakuna matata'
    }
  ]
));
```

Subscribing will automatically unsubscribe from existing subscriptions.

#### Unsubscribe

> Action: `0xb`

Format:
```js
[ 0xb ]
```

## Update participations

#### Indexed Metadata fields

> Action: `0x1`

Format:
```js
[
  0x1,
  {
    group: "[group]",
    type: "[type]",
    name: "[name]",
    anonymous: true,
  }
]
```

#### Workout status

> Action: `0x2`

This list describes the individual participation's workout/interval states in order shown below.

Please always send all the data otherwise the item will be zeroed out.

Format:
```js
[
  0x2, // workout update action
  [
    0, // ElapsedTime (seconds)
    0, // Distance (meters)
    0, // Pace (seconds)
    0, // Watts
    0, // Cal/Hr
    0, // Stroke Rate
    0, // Heart Rate
    0, // Workout State
    0, // Workout Type
    0, // Workout Duration (calories, meters, seconds)
    0, // Workout Duration Type
    0, // Interval Count
    0, // Projected Time (seconds)
    0, // Projected Distance (meters)
  ]
]
```

#### Location updates

> Action: `0x3`

This list describes the individual participation's location

Format:
```js
[
  0x3, // location update action
  [
    0, // Longitude
    0, // Latitude
  ]
]
```







