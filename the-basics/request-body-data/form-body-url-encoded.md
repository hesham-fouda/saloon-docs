# Form Body (URL Encoded)

To get started, make change your method to **POST, PUT or PATCH** depending on the requirements of the API. After that, you will need to add the `HasBody` interface to your request. This interface is required as it tells Saloon to look for a `body()` method supplied by one of the body traits. Without this interface, Saloon will not send any request body to the HTTP client.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;

<strong>class CreateServerRequest extends Request implements HasBody
</strong>{
    protected Method $method = Method::POST;
}
</code></pre>

Next, you will need to add the `HasFormBody` trait to your request. This trait will implement the `body()` method that the `HasBody` interface requires. It also provides a method `defaultBody()` which you can extend to provide a default body on your request.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
<strong>use Saloon\Traits\Body\HasFormBody;
</strong>
class CreateServerRequest extends Request implements HasBody
{
<strong>    use HasFormBody;
</strong>
    protected Method $method = Method::POST;
}
</code></pre>

{% hint style="info" %}
Saloon will automatically send the `Content-Type: application/x-www-form-urlencoded` header for you when using the `HasFormBody` trait, however, if you would like to overwrite this behaviour then you can use the `defaultHeaders` method on the request or modify the headers before the request is sent.
{% endhint %}

### Default Body

There are a couple of ways to interact with the request body to prepare it to be sent. You can either use the methods mentioned below to add to the body on any given instance or you can use the `defaultBody` method on your request. This is recommended because you could then define any requirements as constructor arguments in your request and then standardise your request even more.&#x20;

```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasFormBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasFormBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected string $ubuntuVersion,
        protected string $type,
        protected string $provider
    ){}
    
    protected function defaultBody(): array
    {
        return [
            'ubuntu_version' => $this->ubuntuVersion,
            'type' => $this->type,
            'provider' => $this->provider,
        ];
    }
}
```

Sometimes there can be too many items to define as constructor arguments. If this is the case for you, you can require the user to provide an array, or even a DTO to populate the default body of your request.

{% tabs %}
{% tab title="Using an array" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasFormBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasFormBody;

    protected Method $method = Method::POST;
    
    public function __construct(
        protected array $payload
    ){}
    
    protected function defaultBody(): array
    {
        return $this->payload;
    }
}
```
{% endtab %}

{% tab title="Using a DTO (Data Transfer Object)" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasFormBody;

class CreateServerRequest extends Request implements HasBody
{
    use HasFormBody;

    protected Method $method = Method::POST;
    
    public function __construct(
<strong>        protected Server $server
</strong>    ){}
    
    protected function defaultBody(): array
    {
        return [
            'ubuntu_version' => $this->server->ubuntuVersion,
            'type' => $this->server->type,
            'provider' => $this->server->provider,
        ];
    }
}
</code></pre>
{% endtab %}
{% endtabs %}

### Interacting with the body() method

While you can define the default body on your request, it might be useful to add or modify the body at runtime on a per-request basis. Saloon has the following methods to allow you to modify the form request body:

* add(string $key, mixed $value) -> **Add items to the form body**
* remove(string $key) -> **Remove items from the form body**
* merge(…$values) -> **Merge another array of items into the form body**
* set(array $value) -> **Overwrite the form body entirely**
* all(): array -> **Get all the values of the form body**
* isEmpty(): bool  -> **Check if the body is empty**
* isNotEmpty(): bool -> **Check if the body is not empty**

```php
<?php

$request = new CreateServerRequest;

$request->body()->add('ubuntu_version', '22.04');

$request->body()->merge([
    'type' => 'app',
    'provider' => 'ocean2',
]);

$body = $request->body()->all();

// array: [
//    'ubuntu_version' => '22.04',
//    'type' => 'app',
//    'provider' => 'ocean2',
// ]

```

### Connector Body

If you would like to also have form body on your connector, you can add the same interface and trait to your connector. If you have the trait on both the connector and the request, the properties will be merged. This is useful if you want to have a shared form body across every request, like an authentication token.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Contracts\Body\HasBody;
use Saloon\Traits\Body\HasFormBody;

class ForgeConnector extends Connector implements HasBody
{
    use HasFormBody;

    protected function defaultBody(): array
    {
        return [
            'name' => 'Sam',
        ];
    }
}
```
