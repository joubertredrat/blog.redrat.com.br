---
title: "Validating requests on Symfony Framework"
datePublished: Wed Jul 19 2023 13:16:07 GMT+0000 (Coordinated Universal Time)
cuid: clk9qyham000009mk4iy1aw8i
slug: validating-requests-on-symfony-framework
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/6RFzONgPyl0/upload/b72524adf606b0986011c21399227405.jpeg
tags: opensource, php, symfony, php-frameworks, php8

---

Today [Symfony](https://symfony.com/) is one of the most mature PHP frameworks in the world and because of this, it's used in various projects, including the APIs creation. Recently Symfony included various cool features, like [mapping request data to typed objects](https://symfony.com/blog/new-in-symfony-6-3-mapping-request-data-to-typed-objects), that appeared in version 6.3.

With this, we will take advantage of some of the best resources from the last versions of PHP, that provide support to [attributes](https://www.php.net/releases/8.0/en.php#attributes) and [readonly properties](https://www.php.net/releases/8.1/en.php#readonly_properties) and create validations for Requests in Symfony.

For this, We will use the [Symfony Validation](https://symfony.com/doc/6.3/validation.html) component.

### I have no patience, show me the code!

Okay okay! If you aren't patient to read this post, I have a test project with this implementation of this post in the link below.

[https://github.com/joubertredrat/symfony-request-validation](https://github.com/joubertredrat/symfony-request-validation)

### Basic example

Following the Symfony documentation, we just need to create a class that will be used for mapping the values from the request, as example below.

```php
<?php declare(strict_types=1);

namespace App\Dto;

use App\Validator\CreditCard;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Constraints\Positive;
use Symfony\Component\Validator\Constraints\Range;
use Symfony\Component\Validator\Constraints\Type;

class CreateTransactionDto
{
    public function __construct(
        #[NotBlank(message: 'I dont like this field empty')]
        #[Type('string')]
        public readonly string $firstName,

        #[NotBlank(message: 'I dont like this field empty')]
        #[Type('string')]
        public readonly string $lastName,

        #[NotBlank()]
        #[Type('string')]
        #[CreditCard()]
        public readonly string $cardNumber,

        #[NotBlank()]
        #[Positive()]
        public readonly int $amount,

        #[NotBlank()]
        #[Type('int')]
        #[Range(
            min: 1,
            max: 12,
            notInRangeMessage: 'Expected to be between {{ min }} and {{ max }}, got {{ value }}',
        )]
        public readonly int $installments,

        #[Type('string')]
        public ?string $description = null,
    ) {
    }
}
```

With this, we just use the class as a dependency in the method of the controller with the annotation `#[MapRequestPayload]` and that's it, the values will be automatically mapped into the object, like the example below.

```php
<?php declare(strict_types=1);

namespace App\Controller;

use App\Dto\CreateTransactionDto;
use DateTimeImmutable;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpKernel\Attribute\MapRequestPayload;
use Symfony\Component\Routing\Annotation\Route;

class TransactionController extends AbstractController
{
    #[Route('/api/v1/transactions', name: 'app_api_create_transaction_v1', methods: ['POST'])]
    public function v1Create(#[MapRequestPayload] CreateTransactionDto $createTransaction): JsonResponse
    {
        return $this->json([
            'response' => 'ok',
            'datetime' => (new DateTimeImmutable('now'))->format('Y-m-d H:i:s'),
            'firstName' => $createTransaction->firstName,
            'lastName' => $createTransaction->lastName,
            'amount' => $createTransaction->amount,
            'installments' => $createTransaction->installments,
            'description' => $createTransaction->description,
        ]);
    }
}
```

With this, just do the request and check the result.

```bash
curl --request POST \
  --url http://127.0.0.1:8001/api/v1/transactions \
  --header 'Content-Type: application/json' \
  --data '{
  "firstName": "Joubert",
  "lastName": "RedRat",
  "cardNumber": "4130731304267489",
  "amount": 35011757,
  "installments": 2
}'
```

```json
< HTTP/1.1 200 OK
< Content-Type: application/json

{
  "response": "ok",
  "datetime": "2023-07-04 19:36:37",
  "firstName": "Joubert",
  "lastName": "RedRat",
  "cardNumber": "4130731304267489",
  "amount": 35011757,
  "installments": 2,
  "description": null
}
```

In the example above, if the values aren't correctly filled as the rules defined, will throw an exception and we will receive a response with the found errors.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688500925136/9122def6-7ee6-4799-8ba5-31404918ee07.png?auto=compress,format&format=webp align="center")

The problem is this exception is the default `ValidationFailedException` but as we are building an API, it's necessary a response in JSON format.

With this in mind, we can try a different approach, which will be explained ahead.

**UPDATE August/2023**: After publishing this post and sharing it on Symfony's Slack. **Faizan** and **mdeboer** told me that it's possible to have a JSON response because Symfony has a normalizer for these cases.

To be able to get a JSON response, you should add a header `Accept: application/json`, with this you will get a JSON response, like the example below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691404835024/fa2bcbb4-ca06-4e30-9773-ca09b643c294.png?auto=compress,format&format=webp align="center")

As you can view, the response is in JSON, but, in the Symfony layout. In the next steps, we will do a class in which we can format a JSON response in the layout that we want.

Thanks **Faizan** and **mdeboer** for the contribution**.**

### Abstract Request class

One of the biggest advantages of Symfony is the big and extensive support to DIP "[Dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle)" by your powerful dependency injection container with autowire support.

With this, we will create our abstract class, which will have all code responsible to do the parse of request and validation, as in the example below.

```php
<?php declare(strict_types=1);

namespace App\Request;

use Jawira\CaseConverter\Convert;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\RequestStack;
use Symfony\Component\Validator\Validator\ValidatorInterface;

abstract class AbstractJsonRequest
{
    public function __construct(
        protected ValidatorInterface $validator,
        protected RequestStack $requestStack,
    ) {
        $this->populate();
        $this->validate();
    }

    public function getRequest(): Request
    {
        return $this->requestStack->getCurrentRequest();
    }

    protected function populate(): void
    {
        $request = $this->getRequest();
        $reflection = new \ReflectionClass($this);

        foreach ($request->toArray() as $property => $value) {
            $attribute = self::camelCase($property);
            if (property_exists($this, $attribute)) {
                $reflectionProperty = $reflection->getProperty($attribute);
                $reflectionProperty->setValue($this, $value);
            }
        }
    }

    protected function validate(): void
    {
        $violations = $this->validator->validate($this);
        if (count($violations) < 1) {
            return;
        }

        $errors = [];

        /** @var \Symfony\Component\Validator\ConstraintViolation */
        foreach ($violations as $violation) {
            $attribute = self::snakeCase($violation->getPropertyPath());
            $errors[] = [
                'property' => $attribute,
                'value' => $violation->getInvalidValue(),
                'message' => $violation->getMessage(),
            ];
        }

        $response = new JsonResponse(['errors' => $messages], 400);
        $response->send();
        exit;
    }

    private static function camelCase(string $attribute): string
    {
        return (new Convert($attribute))->toCamel();
    }

    private static function snakeCase(string $attribute): string
    {
        return (new Convert($attribute))->toSnake();
    }
}
```

In the class above it's possible to see the `ValidatorInterface` and the `RequestStack` as the dependencies and in the constructor is executed the fill and validation of attributes.

Also, it's possible to see the conversion between the style `snake_case` and `camelCase` in the attributes and errors, this happens because exists a convention that the fields in the JSON must be `snake_case`, and [PSR-2](https://www.php-fig.org/psr/psr-2/) and [PSR-12](https://www.php-fig.org/psr/psr-12/) suggest the `camelCase` for the names of attributes in the classes, then it's necessary a conversion. For this was used the [Case converter](https://github.com/jawira/case-converter) lib.

But, it's good to remember it isn't an absolute rule, if you want to use a different default from `snake_case` in the JSON, you can.

### Request Class with validation attributes

With an abstract class responsible for all validation, now we will create validation classes, as an example below.

```php
<?php declare(strict_types=1);

namespace App\Request;

use App\Validator\CreditCard;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Constraints\Positive;
use Symfony\Component\Validator\Constraints\Range;
use Symfony\Component\Validator\Constraints\Type;

class CreateTransactionRequest extends AbstractJsonRequest
{
    #[NotBlank(message: 'I dont like this field empty')]
    #[Type('string')]
    public readonly string $firstName;

    #[NotBlank(message: 'I dont like this field empty')]
    #[Type('string')]
    public readonly string $lastName;

    #[NotBlank()]
    #[Type('string')]
    #[CreditCard()]
    public readonly string $cardNumber;

    #[NotBlank()]
    #[Positive()]
    public readonly int $amount;

    #[NotBlank()]
    #[Type('int')]
    #[Range(
        min: 1,
        max: 12,
        notInRangeMessage: 'Expected to be between {{ min }} and {{ max }}, got {{ value }}',
    )]
    public readonly int $installments;

    #[Type('string')]
    public ?string $description = null;
}
```

The big advantage of the class above is that all required attributes of request are with `readonly` status, being possible to warranty the immutability of the data.

Another interesting point is to be able to use attributes of Symfony Validation for doing the necessary validations or even create customized validations.

### Using request class in Route

With the request class ready, now it's just to use as a dependency in the route that we want to do the validation, and it's good to remember that different from the previous example, here isn't necessary `#[MapRequestPayload]` annotation, as an example below.

```php
<?php declare(strict_types=1);

namespace App\Controller;

use App\Request\CreateTransactionRequest;
use DateTimeImmutable;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;

class TransactionController extends AbstractController
{
    #[Route('/api/v2/transactions', name: 'app_api_create_transaction_v2', methods: ['POST'])]
    public function v2Create(CreateTransactionRequest $request): JsonResponse
    {
        return $this->json([
            'response' => 'ok',
            'datetime' => (new DateTimeImmutable('now'))->format('Y-m-d H:i:s'),
            'first_name' => $request->firstName,
            'last_name' => $request->lastName,
            'amount' => $request->amount,
            'installments' => $request->installments,
            'description' => $request->description,
            'headers' => [
                'Content-Type' => $request
                    ->getRequest()
                    ->headers
                    ->get('Content-Type')
                ,
            ],
        ]);
    }
}
```

In the controller above, it's possible to see that we didn't use the traditional `Request` from HttpFoundation, instead of this, our class `CreateTransactionRequest` as dependency and it's in that magic happens because all class dependencies are injected and validation is done.

### Advantages of this approach

In comparison with the basic example, this approach has 2 advantages.

* It's possible to customize the JSON structure and status code of the response as you want.
    
* It's possible to have access to `Request` class from Symfony which was injected as a dependency, with this it's possible to access any information from a request, as headers for example. In the basic example, this isn't possible unless you also add `Request` class as a dependency in route, which sounds strange, because it has two distinct sources of data in the same request.
    

### It's test time!

With our implementation ready, let's go for the tests.

The request example is with deliberate errors to be able to us see the validation response.

```bash
curl --request POST \
  --url http://127.0.0.1:8001/api/v2/transactions \
  --header 'Content-Type: application/json' \
  --data '{
  "last_name": "RedRat",
  "card_number": "1130731304267489",
  "amount": -4,
  "installments": 16
}'
```

```json
< HTTP/1.1 400 Bad Request
< Content-Type: application/json

{
  "errors": [
    {
      "property": "first_name",
      "value": null,
      "message": "I dont like this field empty."
    },
    {
      "property": "card_number",
      "value": "1130731304267489",
      "message": "Expected valid credit card number."
    },
    {
      "property": "amount",
      "value": -4,
      "message": "This value should be positive."
    },
    {
      "property": "installments",
      "value": 16,
      "message": "Expected to be between 1 and 12, got 16"
    }
  ]
}
```

As we can see, the validation happened with success and fields not filled or with wrong data didn't pass in validation and we got a validation response.

Now, let's do a valid request and see that will have success in response because all fields will be filled as we want.

```bash
curl --request POST \
  --url http://127.0.0.1:8001/api/v2/transactions \
  --header 'Content-Type: application/json' \
  --data '{
  "first_name": "Joubert",
  "last_name": "RedRat",
  "card_number": "4130731304267489",
  "amount": 35011757,
  "installments": 2
}'
```

```json
< HTTP/1.1 200 OK
< Content-Type: application/json

{
  "response": "ok",
  "datetime": "2023-07-01 16:39:48",
  "first_name": "Joubert",
  "last_name": "RedRat",
  "card_number": "4130731304267489",
  "amount": 35011757,
  "installments": 2,
  "description": null
}
```

### Limitations

The optional fields can't be `readonly` because if you want to access the data without initialization, PHP will throw an exception. Then, for now, I'm using normal fields with default values for this case.

I'm still looking for any option as a solution for being able to use `readonly` in optional fields, like using Reflection for example, and I accept suggestions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688230052697/9b1767a9-e713-417d-ba47-5e87768c24d9.png?auto=compress,format&format=webp align="center")

Finally, I let my gratitude to my great friend [Vinicius Dias](https://dias.dev/) who helped me with the revision of this post.