### About
Inspired by https://github.com/Marsup/hapi-mongodb, here's an [ioredis](https://github.com/luin/ioredis) plugin for hapijs that supports multiple connections.

### Notes
- Starting from version 2, this plugin switches to ioredis as the redis client
- Starting from version 1.0.0 this plugin only supports Hapi version 17 and above. If you are using hapijs prior to version 17, please check out version [0.9.11-a](https://github.com/midnightcodr/hapi-redis2/tree/0.9.11-a)

### Options
- decorate: string or boolean, mixed use of different types of decorate settings are not allowed.
- settings: can be either an object,
```
{
  port: 6379,          // Redis port
  host: '127.0.0.1',   // Redis host
  family: 4,           // 4 (IPv4) or 6 (IPv6)
  password: 'auth',
  db: 0
}
```
or a string, 
```
'redis://:auth@127.0.0.1:6379/4'
```

Note prior to v2, the `url` option was used.


### Usage example

```javascript
const Hapi = require('hapi')
const Boom = require('boom')

const launchServer = async function() {
    const clientOpts = {
        settings: 'redis://secret@127.0.0.1:6379/2',
        decorate: true
    }

    const server = new Hapi.Server({ port: 8080 })

    await server.register({
        plugin: require('./lib'),
        options: clientOpts
    })

    server.route({
        method: 'GET',
        path: '/redis/{val}',
        async handler(request) {
            const client = request.redis.client

            try {
                await client.set('hello', request.params.val)
                return {
                    result: 'ok'
                }
            } catch (err) {
                throw Boom.internal('Internal Redis error')
            }
        }
    })

    await server.start()
    console.log(`Server started at ${server.info.uri}`)
}

launchServer().catch(err => {
    console.error(err)
    process.exit(1)
})
   
```

Check out [lib/index.test.js](lib/index.test.js) for more usage examples.

Requirements:

    Hapi>=17

    nodejs>=8
