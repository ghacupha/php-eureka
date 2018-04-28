PHP Eureka Client
=================
A PHP client for Spring Cloud Eureka service registration and discovery.


## Installation
You can install this package easily using [Composer](https://getcomposer.org/
):

`composer require "piwvh/php-eureka"`

## Documentation

### Create Eureka Client
The very first thing you should do is to create an instance of `EurekaClient` using your configuration:
```php
$client = new EurekaClient([
    'eurekaDefaultUrl' => 'http://localhost:8761/eureka',
    'hostName' => 'service.hamid.work',
    'appName' => 'service',
    'ip' => '127.0.0.1',
    'port' => ['8080', true],
    'homePageUrl' => 'http://localhost:8080',
    'statusPageUrl' => 'http://localhost:8080/info',
    'healthCheckUrl' => 'http://localhost:8080/health'
]);
```

List of all available configuration includes:

- `eurekaDefaultUrl` (default: `http://localhost:8761`);
- `hostName`
- `appName`
- `ip`
- `status` (default: `UP`)
- `overriddenStatus` (default: `UNKNOWN`)
- `port`
- `securePort` (default: `['443', false]`)
- `countryId ` (default: `1`)
- `dataCenterInfo` (default: `['com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo', 'MyOwn']`)
- `homePageUrl`
- `statusPageUrl`
- `healthCheckUrl`
- `vipAddress`
- `secureVipAddress`
- `heartbeatInterval` (default: `30`)
- `discoveryStrategy` (default: `RandomStrategy`)

You can also change configuration after creating `EurekaClient` instance, using setter methods:
```php
$client->getConfig()->setAppName("my-service");
```

### Operations
After creating EurekaClient instance, there will be multiple operations you can do:
- **Registration:** register your service instance with Eureka
```php
$client->register();
```

- **De-registration:** de-register your service instance from Eureka
```php
$client->deRegister();
```

- **Heartbeat:** send heartbeat to Eureka, to determine the client is up (one time heartbeat)
```php
$client->heartbeat();
```

You can register your instance and sent periodic heartbeat using `start()` method:
```php
$client->start();
```

With this method, first your service gets registered with Eureka using the
configuration you have provided, then a heartbeat will be sent to the Eureka periodically based
on `heartbeatInterval` config. This interval time can be changed just like any other
configuration item:
```php
$client->getConfig()->setHeartbeatInterval(60); // 60 seconds
``` 

- **Service Discovery**: fetch instance of other services from Eureka:
```php
$instance = $client->fetchInstance("another-service");
$homePageUrl = $instance->homePageUrl;
```

### Discovery Strategy
When fetching instances of a service from Eureka, you probably get a list of available
instances registered with Eureka. You can choose one of them based on your desired strategy
of load balancing. For example, a Round-robin or a Random strategy might be your choice.

Currently this library only supports `RandomStrategy`, but you can create your custom
strategy by implementing `getInstance()` method of `DiscoveryStrategy` interface:
```php
class RoundRobinStrategy implements DiscoveryStrategy {
    
    public function getInstance($instances) {
        // return an instance
    }
    
}
```

Then all you have to do is to introduce your custom strategy to `EurekaClient` instance:
```php
$client->getConfig()->setDiscoveryStrategy(new RoundRobinStrategy());
```